#  一级标题 - 测试
##  二级标题 - test
###  三级标题 - 测试
####  四级标题 - test

---

**加粗文本** - test  
*斜体文本* - 测试  
~~删除线文本~~ - test  
**_加粗+斜体_** - 测试  
`行内代码` - test

---

[链接到 Google (测试)](https://google.com)  
[锚点跳转](#图片测试)  
<fake@test.com>  
https://autolink-test.com

---

> 引用块 - test  
> 第二行引用 - 测试

---

无序列表:
- 测试项目 1
- test 项目 2
  - 嵌套测试
    * 嵌套 test

有序列表:
1. 第一条 test
2. 第二条 测试
   1. 嵌套有序 test
   2. 嵌套有序 测试

任务列表:
- [x] 已完成 test
- [ ] 待办 测试

---

表格标题 | 表头测试 | test
--- | --- | ---
行1列1 | 测试数据 | test
行2列1 | **加粗测试** | `代码test`
*斜体test* | ~~删除测试~~ | 100%

---

` ` `python
# 代码块测试
def test():
    print("Hello 测试!")
    return True
` ` ` 
（实际使用时请移除 ` ` ` 中间的空格）

---

<a name="图片测试"></a>
![替代文字-test](https://via.placeholder.com/150?text=图片测试)

---

数学公式 (GitHub 需安装 MathJax 插件):
$$
f(x) = x^2 + \text{测试}
$$

---

脚注测试[1](@ref)  
[1](@ref): 这是脚注内容 - test

---

转义字符测试:  
\*非斜体\*  
\[非链接\]  
\# 非标题
