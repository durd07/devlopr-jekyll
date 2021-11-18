---
title: Insmod
---

## insmod code from busybox
```c
# modutils/insmod.c

int insmod_main(int argc UNUSED_PARAM, char **argv)
{
	char *filename;
	int rc;
	
	filename = *++argv;
	rc = bb_init_module(filename, parse_cmdline_module_options(argv, /*quote_spaces:*/ 0));
	return rc;
}

int FAST_FUNC bb_init_module(const char *filename, const char *options)
{
	size_t image_size;
	char *image;
	int rc;
	bool mmaped;

	if (!options)
		options = "";

	/*
	 * First we try finit_module if available.  Some kernels are configured
	 * to only allow loading of modules off of secure storage (like a read-
	 * only rootfs) which needs the finit_module call.  If it fails, we fall
	 * back to normal module loading to support compressed modules.
	 */
# ifdef __NR_finit_module
	{
		int fd = open(filename, O_RDONLY | O_CLOEXEC);
		if (fd >= 0) {
			rc = finit_module(fd, options, 0) != 0;
			close(fd);
			if (rc == 0)
				return rc;
		}
	}
# endif

	image_size = INT_MAX - 4095;
	mmaped = 0;
	image = try_to_mmap_module(filename, &image_size);

	errno = 0;
	init_module(image, image_size, options);
	return rc;
}

#define init_module(mod, len, opts) syscall(__NR_init_module, mod, len, opts)
This will invoke the syscall in linux kernel
```

in user space, mmap the module ko code into process then invoke SYSCALL init_module


## Linux Kernel
```c
# Code in linux kernel
SYSCALL_DEFINE3(init_module, void __user *, umod,
		unsigned long, len, const char __user *, uargs)
{
	int err;
	struct load_info info = { };

	err = may_init_module();    // CAPACITY CHECK
	if (err)
		return err;

	pr_debug("init_module: umod=%p, len=%lu, uargs=%p\n",
	       umod, len, uargs);

	err = copy_module_from_user(umod, len, &info);    // Load KO into info->hdr
	if (err)
		return err;

	return load_module(&info, uargs, 0);
}

/* Allocate and load the module: note that size of section 0 is always
   zero, and we rely on this for optional sections. */
static int load_module(struct load_info *info, const char __user *uargs,
		       int flags)
{
	struct module *mod;
	long err = 0;
	char *after_dashes;

	err = elf_header_check(info);
	if (err)
		goto free_copy;

	err = setup_load_info(info, flags);
	if (err)
		goto free_copy;

	if (blacklisted(info->name)) {
		err = -EPERM;
		goto free_copy;
	}

	err = module_sig_check(info, flags);
	if (err)
		goto free_copy;

	err = rewrite_section_headers(info, flags);
	if (err)
		goto free_copy;

	/* Check module struct version now, before we try to use module. */
	if (!check_modstruct_version(info, info->mod)) {
		err = -ENOEXEC;
		goto free_copy;
	}

	/* Figure out module layout, and allocate all the memory. */
	mod = layout_and_allocate(info, flags);
	if (IS_ERR(mod)) {
		err = PTR_ERR(mod);
		goto free_copy;
	}

	audit_log_kern_module(mod->name);

	/* Reserve our place in the list. */
	err = add_unformed_module(mod);
	if (err)
		goto free_module;

#ifdef CONFIG_MODULE_SIG
	mod->sig_ok = info->sig_ok;
	if (!mod->sig_ok) {
		pr_notice_once("%s: module verification failed: signature "
			       "and/or required key missing - tainting "
			       "kernel\n", mod->name);
		add_taint_module(mod, TAINT_UNSIGNED_MODULE, LOCKDEP_STILL_OK);
	}
#endif

	/* To avoid stressing percpu allocator, do this once we're unique. */
	err = percpu_modalloc(mod, info);
	if (err)
		goto unlink_mod;

	/* Now module is in final location, initialize linked lists, etc. */
	err = module_unload_init(mod);
	if (err)
		goto unlink_mod;

	init_param_lock(mod);

	/* Now we've got everything in the final locations, we can
	 * find optional sections. */
	err = find_module_sections(mod, info);
	if (err)
		goto free_unload;

	err = check_module_license_and_versions(mod);
	if (err)
		goto free_unload;

	/* Set up MODINFO_ATTR fields */
	setup_modinfo(mod, info);

	/* Fix up syms, so that st_value is a pointer to location. */
	err = simplify_symbols(mod, info);
	if (err < 0)
		goto free_modinfo;

	err = apply_relocations(mod, info);
	if (err < 0)
		goto free_modinfo;

	err = post_relocation(mod, info);
	if (err < 0)
		goto free_modinfo;

	flush_module_icache(mod);

	/* Now copy in args */
	mod->args = strndup_user(uargs, ~0UL >> 1);
	if (IS_ERR(mod->args)) {
		err = PTR_ERR(mod->args);
		goto free_arch_cleanup;
	}

	dynamic_debug_setup(mod, info->debug, info->num_debug);

	/* Ftrace init must be called in the MODULE_STATE_UNFORMED state */
	ftrace_module_init(mod);

	/* Finally it's fully formed, ready to start executing. */
	err = complete_formation(mod, info);
	if (err)
		goto ddebug_cleanup;

	err = prepare_coming_module(mod);
	if (err)
		goto bug_cleanup;

	/* Module is ready to execute: parsing args may do that. */
	after_dashes = parse_args(mod->name, mod->args, mod->kp, mod->num_kp,
				  -32768, 32767, mod,
				  unknown_module_param_cb);
	if (IS_ERR(after_dashes)) {
		err = PTR_ERR(after_dashes);
		goto coming_cleanup;
	} else if (after_dashes) {
		pr_warn("%s: parameters '%s' after `--' ignored\n",
		       mod->name, after_dashes);
	}

	/* Link in to sysfs. */
	err = mod_sysfs_setup(mod, info, mod->kp, mod->num_kp);
	if (err < 0)
		goto coming_cleanup;

	if (is_livepatch_module(mod)) {
		err = copy_module_elf(mod, info);
		if (err < 0)
			goto sysfs_cleanup;
	}

	/* Get rid of temporary copy. */
	free_copy(info);

	/* Done! */
	trace_module_load(mod);

	return do_init_module(mod);    /////////////////////////////

 sysfs_cleanup:
	mod_sysfs_teardown(mod);
 coming_cleanup:
	mod->state = MODULE_STATE_GOING;
	destroy_params(mod->kp, mod->num_kp);
	blocking_notifier_call_chain(&module_notify_list,
				     MODULE_STATE_GOING, mod);
	klp_module_going(mod);
 bug_cleanup:
	/* module_bug_cleanup needs module_mutex protection */
	mutex_lock(&module_mutex);
	module_bug_cleanup(mod);
	mutex_unlock(&module_mutex);

 ddebug_cleanup:
	ftrace_release_mod(mod);
	dynamic_debug_remove(mod, info->debug);
	synchronize_rcu();
	kfree(mod->args);
 free_arch_cleanup:
	module_arch_cleanup(mod);
 free_modinfo:
	free_modinfo(mod);
 free_unload:
	module_unload_free(mod);
 unlink_mod:
	mutex_lock(&module_mutex);
	/* Unlink carefully: kallsyms could be walking list. */
	list_del_rcu(&mod->list);
	mod_tree_remove(mod);
	wake_up_all(&module_wq);
	/* Wait for RCU-sched synchronizing before releasing mod->list. */
	synchronize_rcu();
	mutex_unlock(&module_mutex);
 free_module:
	/* Free lock-classes; relies on the preceding sync_rcu() */
	lockdep_free_key_range(mod->core_layout.base, mod->core_layout.size);

	module_deallocate(mod, info);
 free_copy:
	free_copy(info);
	return err;
}


```
