# Laravel-Mix 图片配置#

### 问题 ###

在vue组件里面的图片,不会生成urlbase64格式,还有目录不是想要的,还没有版本号.

### 解决办法 ###

修改node_modules/laravel-mix/src/builder/webpack-rules.js

    rules.push({
        test: /\.(png|jpe?g|gif)$/,
        loaders: [
            {
                loader: 'url-loader',
                options: {
                    name: path => {
                        if (! /node_modules|bower_components/.test(path)) {
                            return Config.fileLoaderDirs.images + '/[name].[ext]?[hash]';
                        }

                        return Config.fileLoaderDirs.images + '/vendor/' + path
                            .replace(/\\/g, '/')
                            .replace(
                                /((.*(node_modules|bower_components))|images|image|img|assets)\//g, ''
                            ) + '?[hash]';
                    },
                    publicPath: Config.resourceRoot
                }
            },

            {
                loader: 'img-loader',
                options: Config.imgLoaderOptions
            }
			
        ]
    });

    修改为

    rules.push({
        test: /\.(png|jpe?g|gif)$/,
		loader:'url-loader',
		options:{
			limit:8192,
			name:path=>{
				var i1 = path.indexOf('assets')+7;
				var i2 = path.indexOf('\\',i1);
               return Config.fileLoaderDirs.images + '/'+ path.substring(i1,i2) +'/'+'[name].[hash:8].[ext]';
			},
			 publicPath: Config.resourceRoot
		}
    });