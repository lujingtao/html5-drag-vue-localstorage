# 前言
因业务需求，需要在页面上列出多个视频，并能拖放排序，且记录排序。
现使用技术如下：
vue + html5的drag事件实现元素拖放排序 +l ocalStorage记录到本地浏览器上，刷新页面不会重置排序。

本示例为了更好的用户体验，排序后使用css3 的 translate 来变更元素位置，而不是2个元素直接交换内容！如果直接交换内容，则视频会重新加载，用户体验不好！

# 效果
实现效果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200116163802168.gif)

在线演示地址：

# 技术要点
1、通过 `draggable="true"` 开启元素的排序功能，例如：`<span draggable="true"><span>`；
然后绑定`dragstart、dragenter、dragover...`等事件，具体可以自行百度，其中本例也详细列出事件触发情况，按f12查看输出内容即可：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200116163812645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lhbWx1amluZ3Rhbw==,size_16,color_FFFFFF,t_70)

2、程序开始时从localStorage里面取排序记录，如果有则赋值给当前列表项目，其中还要处理列表项目变更情况（因为真实情况下，列表项目是从数据库读取的）；

```js
            //如果localStorage里面已有排序记录则赋值 this.sorts
            let sorts = []; //本地排序记录集合
            if (!!window.localStorage) {
                let listSort = window.localStorage["listSort"];
                if (listSort != undefined) {
                    sorts = JSON.parse(listSort);
                }
            }

            //如果有排序记录则设置排序，没有则顺序排序
            if (sorts.length == 0) {
                this.items.forEach((item, index) => {
                    item.sort = index;
                });
            } else {
                this.items.forEach((item) => {
                    for (let i = 0; i < sorts.length; i++) {
                        if (item.id == sorts[i].id) {
                            item.sort = sorts[i].sort;
                            break;
                        }
                        else if( i==sorts.length-1 ){
                            item.sort=sorts.length;
                        }
                    }
                });

                this.updateSort();
            }
```


3、当项目元素变更（删除项目）后，vue会更新项目（items）再更新项目元素（li），所以li的位置变更需要在vue的updated里面：

```js
        updated() {
            //当项目集合变更后，触发li元素变更，再执行li的位置变更
            this.updateItemsSort();
        },
```

