---
title: Study
---
# Study

C++ class default methods
```c++
#include <iostream>
using namespace std;

class Test
{
private:
public:
    // 默认构造函数;
    Test() { std::cout << "Test" << std::endl; }
    // 默认析构函数
    ~Test() { std::cout << "~Test" << std::endl; }
    // 默认拷贝构造函数
    Test(const Test& t) { std::cout << "Copy Test" << std::endl; }
    // 默认重载赋值运算符函数
    Test& operator = (const Test&) { std::cout << "Assignment" << std::endl; }
    // 默认重载取址运算符函数
    Test* operator & () { std::cout << "Address" << std::endl; }
    // 默认重载取址运算符const函数
    const Test* operator & () const { std::cout << "Const Addr" << std::endl; }
    // 默认移动构造函数
    Test(Test&&) { std::cout << "Move" << std::endl; }
    // 默认重载移动赋值操作符
    Test& operator = (const Test&&) { std::cout << "Move Assign" << std::endl; }
};

Test fun(void)
{
    std::cout << __func__ << " Enter" << std::endl;
    Test t;
    std::cout << __func__ << " Leave" << std::endl;
    return t;
}

int main(int argc, char* argv[])
{
    std::cout << __func__ << " Enter" << std::endl;
    auto resp = fun();
    std::cout << __func__ << " Leave" << std::endl;
    return 0;
}
```

C++ 20 Modules
```

```

## get registry tag list
curl -sL -u restapi:restapi -X GET https://ntas-docker-candidates.repo.lab.pl.alcatel-lucent.com/v2/opentas/os-base/tags/list

## git show the branch introduced commit
```
git name-rev commit-id
```

## GTest
1. install
```sh
dnf install gtest gtest-devel
```

2. Test it
```c
#include<gtest/gtest.h>
int add(int a,int b){
    return a+b;
}
TEST(testCase,test0){
    EXPECT_EQ(add(2,3),5);
}
int main(int argc,char **argv){
  testing::InitGoogleTest(&argc,argv);
  return RUN_ALL_TESTS();
}
```

3. compile
```sh
g++ test.cc -lgtest -lpthread
./a.out
```

## Use beautiful mindmap graph in my pages
There is a tool [https://markmap.js.org/](https://markmap.js.org/) can be used to generate SVG from markdown mindmap syntax which can be embedded into html, and also a plugin named `markmap` in VS Code. it can be used to export html file. the html also can be generated from [https://markmap.js.org/repl](https://markmap.js.org/repl)

copy below into `head.html`
```html
<style>
* {
  margin: 0;
  padding: 0;
}
#mindmap {
  display: block;
  width: 100vw;
  height: 100vh;
}
</style>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/prismjs@1/themes/prism.css"><link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.css">
```

and copy svg into markdown file
```html
<svg id="mindmap"></svg>
<script src="https://cdn.jsdelivr.net/npm/d3@6.2.0"></script><script src="https://cdn.jsdelivr.net/npm/markmap-view@0.1.1"></script><script>(e=>{e().registerRefreshPromise(new Promise((e=>{window.WebFontConfig={custom:{families:["KaTeX_AMS","KaTeX_Caligraphic:n4,n7","KaTeX_Fraktur:n4,n7","KaTeX_Main:n4,n7,i4,i7","KaTeX_Math:i4,i7","KaTeX_Script","KaTeX_SansSerif:n4,n7,i4","KaTeX_Size1","KaTeX_Size2","KaTeX_Size3","KaTeX_Size4","KaTeX_Typewriter"]},active:()=>{e()}}})))})(()=>window.markmap)</script><script src="https://cdn.jsdelivr.net/npm/webfontloader@1.6.28/webfontloader.js" defer></script><script>((e,t)=>{const{Markmap:r}=e();window.mm=r.create("svg#mindmap",null,t)})(()=>window.markmap,{"t":"heading","d":1,"p":{},"v":"markmap","c":[{"t":"heading","d":2,"p":{},"v":"Links","c":[{"t":"list_item","d":4,"p":{},"v":"<a href=\"https://markmap.js.org/\">https://markmap.js.org/</a>"},{"t":"list_item","d":4,"p":{},"v":"<a href=\"https://github.com/gera2ld/markmap\">GitHub</a>"}]},{"t":"heading","d":2,"p":{},"v":"Related","c":[{"t":"list_item","d":4,"p":{},"v":"<a href=\"https://github.com/gera2ld/coc-markmap\">coc-markmap</a>"},{"t":"list_item","d":4,"p":{},"v":"<a href=\"https://github.com/gera2ld/gatsby-remark-markmap\">gatsby-remark-markmap</a>"}]},{"t":"heading","d":2,"p":{},"v":"Features","c":[{"t":"list_item","d":4,"p":{},"v":"links"},{"t":"list_item","d":4,"p":{},"v":"<strong>inline</strong> <del>text</del> <em>styles</em>"},{"t":"list_item","d":4,"p":{},"v":"multiline\ntext"},{"t":"list_item","d":4,"p":{},"v":"<code>inline code</code>"},{"t":"list_item","d":4,"p":{},"v":"<pre><code class=\"language-js\">console<span class=\"token punctuation\">.</span><span class=\"token function\">log</span><span class=\"token punctuation\">(</span><span class=\"token string\">'code block'</span><span class=\"token punctuation\">)</span><span class=\"token punctuation\">;</span>\n</code></pre>\n"},{"t":"list_item","d":4,"p":{},"v":"Katex - <span class=\"katex\"><span class=\"katex-mathml\"><math xmlns=\"http://www.w3.org/1998/Math/MathML\"><semantics><mrow><mi>x</mi><mo>=</mo><mfrac><mrow><mo>−</mo><mi>b</mi><mo>±</mo><msqrt><mrow><msup><mi>b</mi><mn>2</mn></msup><mo>−</mo><mn>4</mn><mi>a</mi><mi>c</mi></mrow></msqrt></mrow><mrow><mn>2</mn><mi>a</mi></mrow></mfrac></mrow><annotation encoding=\"application/x-tex\">x = {-b \\pm \\sqrt{b^2-4ac} \\over 2a}</annotation></semantics></math></span><span class=\"katex-html\" aria-hidden=\"true\"><span class=\"base\"><span class=\"strut\" style=\"height:0.43056em;vertical-align:0em;\"></span><span class=\"mord mathnormal\">x</span><span class=\"mspace\" style=\"margin-right:0.2777777777777778em;\"></span><span class=\"mrel\">=</span><span class=\"mspace\" style=\"margin-right:0.2777777777777778em;\"></span></span><span class=\"base\"><span class=\"strut\" style=\"height:1.384482em;vertical-align:-0.345em;\"></span><span class=\"mord\"><span class=\"mord\"><span class=\"mopen nulldelimiter\"></span><span class=\"mfrac\"><span class=\"vlist-t vlist-t2\"><span class=\"vlist-r\"><span class=\"vlist\" style=\"height:1.039482em;\"><span style=\"top:-2.6550000000000002em;\"><span class=\"pstrut\" style=\"height:3em;\"></span><span class=\"sizing reset-size6 size3 mtight\"><span class=\"mord mtight\"><span class=\"mord mtight\">2</span><span class=\"mord mathnormal mtight\">a</span></span></span></span><span style=\"top:-3.23em;\"><span class=\"pstrut\" style=\"height:3em;\"></span><span class=\"frac-line\" style=\"border-bottom-width:0.04em;\"></span></span><span style=\"top:-3.3939999999999997em;\"><span class=\"pstrut\" style=\"height:3em;\"></span><span class=\"sizing reset-size6 size3 mtight\"><span class=\"mord mtight\"><span class=\"mord mtight\">−</span><span class=\"mord mathnormal mtight\">b</span><span class=\"mbin mtight\">±</span><span class=\"mord sqrt mtight\"><span class=\"vlist-t vlist-t2\"><span class=\"vlist-r\"><span class=\"vlist\" style=\"height:0.9221171428571429em;\"><span class=\"svg-align\" style=\"top:-3em;\"><span class=\"pstrut\" style=\"height:3em;\"></span><span class=\"mord mtight\" style=\"padding-left:0.833em;\"><span class=\"mord mtight\"><span class=\"mord mathnormal mtight\">b</span><span class=\"msupsub\"><span class=\"vlist-t\"><span class=\"vlist-r\"><span class=\"vlist\" style=\"height:0.7463142857142857em;\"><span style=\"top:-2.786em;margin-right:0.07142857142857144em;\"><span class=\"pstrut\" style=\"height:2.5em;\"></span><span class=\"sizing reset-size3 size1 mtight\"><span class=\"mord mtight\">2</span></span></span></span></span></span></span></span><span class=\"mbin mtight\">−</span><span class=\"mord mtight\">4</span><span class=\"mord mathnormal mtight\">a</span><span class=\"mord mathnormal mtight\">c</span></span></span><span style=\"top:-2.882117142857143em;\"><span class=\"pstrut\" style=\"height:3em;\"></span><span class=\"hide-tail mtight\" style=\"min-width:0.853em;height:1.08em;\"><svg width='400em' height='1.08em' viewBox='0 0 400000 1080' preserveAspectRatio='xMinYMin slice'><path d='M95,702\nc-2.7,0,-7.17,-2.7,-13.5,-8c-5.8,-5.3,-9.5,-10,-9.5,-14\nc0,-2,0.3,-3.3,1,-4c1.3,-2.7,23.83,-20.7,67.5,-54\nc44.2,-33.3,65.8,-50.3,66.5,-51c1.3,-1.3,3,-2,5,-2c4.7,0,8.7,3.3,12,10\ns173,378,173,378c0.7,0,35.3,-71,104,-213c68.7,-142,137.5,-285,206.5,-429\nc69,-144,104.5,-217.7,106.5,-221\nl0 -0\nc5.3,-9.3,12,-14,20,-14\nH400000v40H845.2724\ns-225.272,467,-225.272,467s-235,486,-235,486c-2.7,4.7,-9,7,-19,7\nc-6,0,-10,-1,-12,-3s-194,-422,-194,-422s-65,47,-65,47z\nM834 80h400000v40h-400000z'/></svg></span></span></span><span class=\"vlist-s\">​</span></span><span class=\"vlist-r\"><span class=\"vlist\" style=\"height:0.11788285714285718em;\"><span></span></span></span></span></span></span></span></span></span><span class=\"vlist-s\">​</span></span><span class=\"vlist-r\"><span class=\"vlist\" style=\"height:0.345em;\"><span></span></span></span></span></span><span class=\"mclose nulldelimiter\"></span></span></span></span></span></span>"}]}]})</script>
```
The graph like below:

<svg id="mindmap"></svg>
<script src="https://cdn.jsdelivr.net/npm/d3@6.2.0"></script><script src="https://cdn.jsdelivr.net/npm/markmap-view@0.1.1"></script><script>(e=>{e().registerRefreshPromise(new Promise((e=>{window.WebFontConfig={custom:{families:["KaTeX_AMS","KaTeX_Caligraphic:n4,n7","KaTeX_Fraktur:n4,n7","KaTeX_Main:n4,n7,i4,i7","KaTeX_Math:i4,i7","KaTeX_Script","KaTeX_SansSerif:n4,n7,i4","KaTeX_Size1","KaTeX_Size2","KaTeX_Size3","KaTeX_Size4","KaTeX_Typewriter"]},active:()=>{e()}}})))})(()=>window.markmap)</script><script src="https://cdn.jsdelivr.net/npm/webfontloader@1.6.28/webfontloader.js" defer></script><script>((e,t)=>{const{Markmap:r}=e();window.mm=r.create("svg#mindmap",null,t)})(()=>window.markmap,{"t":"heading","d":1,"p":{},"v":"markmap","c":[{"t":"heading","d":2,"p":{},"v":"Links","c":[{"t":"list_item","d":4,"p":{},"v":"<a href=\"https://markmap.js.org/\">https://markmap.js.org/</a>"},{"t":"list_item","d":4,"p":{},"v":"<a href=\"https://github.com/gera2ld/markmap\">GitHub</a>"}]},{"t":"heading","d":2,"p":{},"v":"Related","c":[{"t":"list_item","d":4,"p":{},"v":"<a href=\"https://github.com/gera2ld/coc-markmap\">coc-markmap</a>"},{"t":"list_item","d":4,"p":{},"v":"<a href=\"https://github.com/gera2ld/gatsby-remark-markmap\">gatsby-remark-markmap</a>"}]},{"t":"heading","d":2,"p":{},"v":"Features","c":[{"t":"list_item","d":4,"p":{},"v":"links"},{"t":"list_item","d":4,"p":{},"v":"<strong>inline</strong> <del>text</del> <em>styles</em>"},{"t":"list_item","d":4,"p":{},"v":"multiline\ntext"},{"t":"list_item","d":4,"p":{},"v":"<code>inline code</code>"},{"t":"list_item","d":4,"p":{},"v":"<pre><code class=\"language-js\">console<span class=\"token punctuation\">.</span><span class=\"token function\">log</span><span class=\"token punctuation\">(</span><span class=\"token string\">'code block'</span><span class=\"token punctuation\">)</span><span class=\"token punctuation\">;</span>\n</code></pre>\n"},{"t":"list_item","d":4,"p":{},"v":"Katex - <span class=\"katex\"><span class=\"katex-mathml\"><math xmlns=\"http://www.w3.org/1998/Math/MathML\"><semantics><mrow><mi>x</mi><mo>=</mo><mfrac><mrow><mo>−</mo><mi>b</mi><mo>±</mo><msqrt><mrow><msup><mi>b</mi><mn>2</mn></msup><mo>−</mo><mn>4</mn><mi>a</mi><mi>c</mi></mrow></msqrt></mrow><mrow><mn>2</mn><mi>a</mi></mrow></mfrac></mrow><annotation encoding=\"application/x-tex\">x = {-b \\pm \\sqrt{b^2-4ac} \\over 2a}</annotation></semantics></math></span><span class=\"katex-html\" aria-hidden=\"true\"><span class=\"base\"><span class=\"strut\" style=\"height:0.43056em;vertical-align:0em;\"></span><span class=\"mord mathnormal\">x</span><span class=\"mspace\" style=\"margin-right:0.2777777777777778em;\"></span><span class=\"mrel\">=</span><span class=\"mspace\" style=\"margin-right:0.2777777777777778em;\"></span></span><span class=\"base\"><span class=\"strut\" style=\"height:1.384482em;vertical-align:-0.345em;\"></span><span class=\"mord\"><span class=\"mord\"><span class=\"mopen nulldelimiter\"></span><span class=\"mfrac\"><span class=\"vlist-t vlist-t2\"><span class=\"vlist-r\"><span class=\"vlist\" style=\"height:1.039482em;\"><span style=\"top:-2.6550000000000002em;\"><span class=\"pstrut\" style=\"height:3em;\"></span><span class=\"sizing reset-size6 size3 mtight\"><span class=\"mord mtight\"><span class=\"mord mtight\">2</span><span class=\"mord mathnormal mtight\">a</span></span></span></span><span style=\"top:-3.23em;\"><span class=\"pstrut\" style=\"height:3em;\"></span><span class=\"frac-line\" style=\"border-bottom-width:0.04em;\"></span></span><span style=\"top:-3.3939999999999997em;\"><span class=\"pstrut\" style=\"height:3em;\"></span><span class=\"sizing reset-size6 size3 mtight\"><span class=\"mord mtight\"><span class=\"mord mtight\">−</span><span class=\"mord mathnormal mtight\">b</span><span class=\"mbin mtight\">±</span><span class=\"mord sqrt mtight\"><span class=\"vlist-t vlist-t2\"><span class=\"vlist-r\"><span class=\"vlist\" style=\"height:0.9221171428571429em;\"><span class=\"svg-align\" style=\"top:-3em;\"><span class=\"pstrut\" style=\"height:3em;\"></span><span class=\"mord mtight\" style=\"padding-left:0.833em;\"><span class=\"mord mtight\"><span class=\"mord mathnormal mtight\">b</span><span class=\"msupsub\"><span class=\"vlist-t\"><span class=\"vlist-r\"><span class=\"vlist\" style=\"height:0.7463142857142857em;\"><span style=\"top:-2.786em;margin-right:0.07142857142857144em;\"><span class=\"pstrut\" style=\"height:2.5em;\"></span><span class=\"sizing reset-size3 size1 mtight\"><span class=\"mord mtight\">2</span></span></span></span></span></span></span></span><span class=\"mbin mtight\">−</span><span class=\"mord mtight\">4</span><span class=\"mord mathnormal mtight\">a</span><span class=\"mord mathnormal mtight\">c</span></span></span><span style=\"top:-2.882117142857143em;\"><span class=\"pstrut\" style=\"height:3em;\"></span><span class=\"hide-tail mtight\" style=\"min-width:0.853em;height:1.08em;\"><svg width='400em' height='1.08em' viewBox='0 0 400000 1080' preserveAspectRatio='xMinYMin slice'><path d='M95,702\nc-2.7,0,-7.17,-2.7,-13.5,-8c-5.8,-5.3,-9.5,-10,-9.5,-14\nc0,-2,0.3,-3.3,1,-4c1.3,-2.7,23.83,-20.7,67.5,-54\nc44.2,-33.3,65.8,-50.3,66.5,-51c1.3,-1.3,3,-2,5,-2c4.7,0,8.7,3.3,12,10\ns173,378,173,378c0.7,0,35.3,-71,104,-213c68.7,-142,137.5,-285,206.5,-429\nc69,-144,104.5,-217.7,106.5,-221\nl0 -0\nc5.3,-9.3,12,-14,20,-14\nH400000v40H845.2724\ns-225.272,467,-225.272,467s-235,486,-235,486c-2.7,4.7,-9,7,-19,7\nc-6,0,-10,-1,-12,-3s-194,-422,-194,-422s-65,47,-65,47z\nM834 80h400000v40h-400000z'/></svg></span></span></span><span class=\"vlist-s\">​</span></span><span class=\"vlist-r\"><span class=\"vlist\" style=\"height:0.11788285714285718em;\"><span></span></span></span></span></span></span></span></span></span><span class=\"vlist-s\">​</span></span><span class=\"vlist-r\"><span class=\"vlist\" style=\"height:0.345em;\"><span></span></span></span></span></span><span class=\"mclose nulldelimiter\"></span></span></span></span></span></span>"}]}]})</script>
