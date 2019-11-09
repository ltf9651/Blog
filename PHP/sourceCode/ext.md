## PHP扩展

1. 创建模板： ./ext_skel --extname=helloworld
1. 去除 config.m4 的部分注释（dnl）
1. 通过 phpize 编译
1. vim helloworld.c

``` PHP
PHP_FUNCTION(confirm_helloworld_compiled)
{
	char *arg = NULL;
	size_t arg_len, len;
	zend_string *strg;

	if (zend_parse_parameters(ZEND_NUM_ARGS(), "s", &arg, &arg_len) == FAILURE) {
		return;
	}

	strg = strpprintf(0, "Congratulations! You have successfully modified ext/%.78s/config.m4. Module %.78s is now compiled into PHP.", "helloworld", arg);

	RETURN_STR(strg);
}
```

1. ./conigure -with-php-config=xxx
1. make
1. ls modules -> helloworld.so  helloworld.la
1. make install 
1. vim php.ini : extension = helloworld.so

``` C
// 核心数据结构
zend_module_entry helloworld_module_entry = {
	STANDARD_MODULE_HEADER,
	"helloworld",
	helloworld_functions,
	PHP_MINIT(helloworld),  // 全局变量设置
	PHP_MSHUTDOWN(helloworld), // 析构全局变量
	PHP_RINIT(helloworld),		// 请求相关变量的定义
	PHP_RSHUTDOWN(helloworld),	// 重置请求
	PHP_MINFO(helloworld),
	PHP_HELLOWORLD_VERSION,
	STANDARD_MODULE_PROPERTIES
};
```

