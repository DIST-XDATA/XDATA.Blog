---
title: Hexo 配置完善
tags: Hexo
categories: hexo
---

hexo搭建后,我们需要加入许多个性化功能,在此进行记录.

<!-- more -->

## 添加返回顶部按钮

1. 打开\themes\landscape\layout\_partial,新建文件名称为totop.ejs.

        <div id="totop">
            <a title="javascript:;">返回<br>顶部</a>
        </div>

2. 打开\themes\landscape\source\js,新建文件totop.js.

        (function($) {
            var upperLimit = 1000;

            // Our scroll link element
            var scrollElem = $('#totop');

            // Scroll to top speed
            var scrollSpeed = 1600;
            scrollElem.hide();
            $(window).scroll(function () {
                var scrollTop = $(document).scrollTop();
                if ( scrollTop > upperLimit ) {
                    $(scrollElem).stop().fadeTo(300, 1); 
                }else{
                    $(scrollElem).stop().fadeTo(300, 0); 
                }
            });
            $(scrollElem).click(function(){
                $('html, body').animate({scrollTop:0}, scrollSpeed); return false;
            });
        })(jQuery);

3. 打开\themes\landscape\layout\_partial\after_footer.ejs,添加代码

        <%- partial('totop') %>
        <script src="<%- config.root %>js/totop.js"></script>

4. 设置样式,打开\themes\landscape\source\css\_partial,创建文件totop.styl,添加样式代码

        #totop
            position:fixed
            bottom:50px
            right:30px
            text-align: center
            z-index: 100
            a
                display: block
                padding 4px 8px
                color: #fff
                background-color: #258fb8
                cursor: pointer
                border-radius: 5px
                &:hover
                    opacity: .8

    打开\themes\landscape\source\css\style.styl,添加代码

            @import "_partial/totop"