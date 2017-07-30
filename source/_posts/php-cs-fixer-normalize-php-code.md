---
title: 关于代码规范
date: 2017-07-30 21:16:52
tags:
- coding style
- php-cs-fixer
---

对于多人协作的项目，如果没有强制性的代码风格约束，很容易出现不一致问题。
我们需要有规范来告诉别人应该怎么去做，但是我们无法保证每个人都是在按照规范来做。
*标准规范都应该落实到强制性约束上，这可以让开发者无需考虑规范，但又始终在规范之内*.
     
这里我不得不赞一下`golang`语言，存在相应的工具，你无需纠结于缩进使用**空格**还是**TAB**, 
 `gofmt`来解决, `import`的顺序是如何，也有`goimports`的工具.
 
之前一直也想着整理PHP代码规范，然后落实到项目中去，但没有具体落实下去。留下来
的技术债，终究是要还的。最近项目集成了[gitlab-ci](https://docs.gitlab.com/ce/ci/), 
可以在提交代码的时候在代码检查，正好符合强制性约束的要求。

一开始使用的是[PHP_Code_Sniffer](https://github.com/squizlabs/PHP_CodeSniffer)来做代码检查，但是因为一些历史原因，不能很好的适配，如果忽略warning级别的提示，检查的力度又太粗了。于是换了[php-cs-fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer), 对里面的一些配置项做了筛选，确定了最终的配置。

<!-- more --> 

配置文件如下:

```php
<?php

return PhpCsFixer\Config::create()
    ->setRiskyAllowed(true)
    ->setRules([
        '@PSR2' => true,
        'array_syntax' => [
            'syntax' => 'short',
        ],
        'binary_operator_spaces' => true,
        'blank_line_before_return' => true,
        'concat_space' => [
            'spacing' => 'one',
        ],
        'function_typehint_space' => true,
        'hash_to_slash_comment' => true,
        'linebreak_after_opening_tag' => true,
        'lowercase_cast' => true,
        'method_separation' => true,
        'native_function_casing' => true,
        'new_with_braces' => true,
        'no_alias_functions' => true,
        'no_blank_lines_after_class_opening' => true,
        'no_blank_lines_after_phpdoc' => true,
        'no_blank_lines_before_namespace' => true,
        'no_empty_comment' => true,
        'no_empty_phpdoc' => true,
        'no_empty_statement' => true,
        'no_extra_consecutive_blank_lines' => [
            'continue',
            'curly_brace_block',
            'extra',
            'parenthesis_brace_block',
            'square_brace_block',
            'throw',
        ],
        'no_leading_import_slash' => true,
        'no_leading_namespace_whitespace' => true,
        'no_multiline_whitespace_around_double_arrow' => true,
        'no_multiline_whitespace_before_semicolons' => true,
        'no_short_bool_cast' => true,
        'no_singleline_whitespace_before_semicolons' => true,
        'no_trailing_comma_in_list_call' => true,
        'no_trailing_comma_in_singleline_array' => true,
        'no_unneeded_control_parentheses' => [
            'break',
            'clone',
            'continue',
            'echo_print',
            'return',
            'switch_case',
        ],
        'no_unreachable_default_argument_value' => true,
        'no_unused_imports' => true,
        'no_useless_else' => true,
        'no_useless_return' => true,
        'no_whitespace_before_comma_in_array' => true,
        'no_whitespace_in_blank_line' => true,
        'normalize_index_brace' => true,
        'ordered_imports' => true,
        'phpdoc_indent' => true,
        'phpdoc_scalar' => true,
        'phpdoc_types' => true,
        'phpdoc_single_line_var_spacing' => true,
        'phpdoc_annotation_without_dot' => true,
        // 'phpdoc_add_missing_param_annotation' => true,
        // 'phpdoc_no_package' => true,
        // 'phpdoc_order' => true,
        'self_accessor' => true,
        'short_scalar_cast' => true,
        'single_quote' => true,
        'standardize_not_equals' => true,
        'ternary_operator_spaces' => true,
        'trailing_comma_in_multiline_array' => true,
        'whitespace_after_comma_in_array' => true,
    ]);
```
主要效果: 

- 去掉无用`use`
- 使用`use`时去掉开始的`\`, 如`use \ApiTester;` => `use ApiTester;`
- 一个`class`文件中的`use`语句按照字母序排序 
- `else if`变成单个`elseif`
- 去掉无用的`else`
- 不使用`array()`, 使用`[]`
- `=>`, `+`, `-`, `.`, `?`操作符两侧各保留一个空格
- `return`语句前保留一个空行
- 不使用函数的别名，如使用 `implode()`, 而不是`join()`
- 函数之间以单个空行分隔
- 纯字符串使用单引号，不使用双引号
 - 使用`new Object();` 而不是 `new Object;`
- 注释不以`.`结尾
- 注释缩进

在**CI**中检测上一次的commit:

```shell
git diff --name-only --diff-filter=ACMRT --exit-code HEAD~1
```
通过先fix: 

```shell
php ci/php-cs-fixer.phar fix --config=.php_cs $file
```
然后比较改动:

```shell
git diff --exit-code --color=always  $file
```

来确定文件是否符合规范并输出不同
