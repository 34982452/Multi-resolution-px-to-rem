# Multi-resolution-px-to-rem

由于前端项目需要满足在各种尺寸的显示设备上（包括手机）进行适配，以前经常出现开发环境还可以，老板们大屏显示时尺寸没有同比例放大的缺陷。

如果用%或者rem的函数来进行适配，适配工作量都非常大，需要每个开发者都关注此功能。

比较好的解决方案是由框架架构自动完成适配，开发者只根据设计稿书写CSS的px值，现将调研好的完整方案分享给你们，适用于各个项目。

能够极大地提高开发效率。


一 CSS文件的适配

    extraPostCSSPlugins: [
        //https://www.npmjs.com/package/postcss-plugin-px2rem
        px2rem({
            rootValue: 13.66,
            propBlackList: ["border-width", "min-width", "min-height"],
            selectorBlackList: ["t_npx"]
        })
    ],

13.66是设计稿宽度/100，然后各个开发者直接将设计稿的px值填写到css文件中，由框架自动完成px到rem的转换

HTML入口文件：

        <script>
            (function(doc, win) {
                var docEl = doc.documentElement,
                    // 判断window中是否有orientationchange方法
                    resizeEvt = "orientationchange" in window ? "orientationchange" : "resize",
                    recalc = function() {
                        var clientWidth = docEl.clientWidth;
                        if (!clientWidth) return;
                        //设置基础html的fontsize
                        docEl.style.fontSize =
                            clientWidth > 1080 ? +((13.66 * clientWidth) / 1366).toFixed(2) + "px" : "10.8px";
                        //console.log(docEl.style.fontSize, "docEl.style.fontSize");
                    };
                if (!doc.addEventListener) return;
                win.addEventListener(resizeEvt, recalc, false);
                doc.addEventListener("DOMContentLoaded", recalc, false);
            })(document, window);
        </script>


二 HTML/JSX文件的适配

可以支持在HTML/JSX中的内敛样式的px单位进行支持

Umi的官方文档对webpack的扩展介绍得不清楚

    chainWebpack(config){
        config.module
            .rule('jsx-px2rem-loader')
            .test(/\.tsx$/)
            .exclude
            .add([path.resolve('../src/pages/.umi'), path.resolve('node_modules')])
            .end()
            .use('../webpackloader/jsx-px2rem-loader')
            .loader(path.join(__dirname, '../webpackloader/jsx-px2rem-loader'));
    }


jsx-px2rem-loader的代码：
    var _ = require("lodash");

    const pxReg = /\b(\d+(\.\d+)?)px\b/g;

    const styleReg = {
        marginTop: /\bmarginTop(?:\s+):(?:\s+)?(\d+)/g,
        marginRight: /\bmarginRight(?:\s+)?:(?:\s+)?(\d+)/g,
        marginBottom: /\bmarginBottom(?:\s+)?:(?:\s+)?(\d+)/g,
        marginLeft: /\bmarginLeft(?:\s+)?:(?:\s+)?(\d+)/g,
        fontSize: /\bfontSize(?:\s+)?:(?:\s+)?(\d+)/g,
        paddingTop: /\bpaddingTop(?:\s+)?:(?:\s+)?(\d+)/g,
        paddingRight: /\bpaddingRight(?:\s+)?:(?:\s+)?(\d+)/g,
        paddingBottom: /\bpaddingBottom(?:\s+)?:(?:\s+)?(\d+)/g,
        paddingLeft: /\bpaddingLeft(?:\s+)?:(?:\s+)?(\d+)/g,
        height: /\bheight(?:\s+)?=(?:\s+)?(\'||\")?(\d+)?=(\'||\")?/g,
        width: /\bwidth(?:\s+)?=(?:\s+)?(\'||\")?(\d+)?=(\'||\")?/g,
    }

    const numReg = /(\d+)/g;

    module.exports = function(source) {
        if (this.cacheable) {
            this.cacheable();
        }
        let backUp = source;

        if (pxReg.test(backUp)) {
            backUp = backUp.replace(pxReg, px => {
                let val = px.replace(numReg, num => {
                    return num / 13.66;
                });
                val = val.replace(/px$/, 'rem');
                return val;
            });
        }


        // // marginRight: 13.66 => marginRight: '1rem'
        _.each(styleReg, (reg, styleName) => {
            if (reg.test(backUp)) {
                backUp = backUp.replace(reg, val => {
                    return val.replace(numReg, num => {
                        return `"${num / 13.66}rem"`;
                    });
                });
            }
        })

        return backUp;
    }

