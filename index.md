## csapp lab2 bomb
### 总览
总共有六个炸弹，需要我们一一拆除。题目提供了可执行文件bomb与bomb.c，.c文件中只提供了主函数。
##phase_1
```markdown
/* Hmm...  Six phases must be more secure than one phase! */
    input = read_line();             /* Get input                   */
    phase_1(input);                  /* Run the phase               */
    phase_defused();                 /* Drat!  They figured it out!
				      * Let me know how they did it. */
printf("Phase 1 defused. How about the next one?\n");

 ```
接受输入后进行判断，判断通过会输出“Phase 1 defused. How about the next one?\n”。为了拆除炸弹，我们需要推理出input是什么。
命令详见书3.10.2
现在终端输入gdb bomb进行gdb调试

## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/1051690662/csapp_lab/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/1051690662/csapp_lab/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
