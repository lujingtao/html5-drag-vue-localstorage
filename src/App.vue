<template>
    <div id="app">
        <h1>html5拖放 + vue + localStorage，实现拖放排序并记录到本地</h1>
        <ul class="list">
            <li v-for="item in items" :key="item.id">
                <span draggable="true" :sort="item.sort">{{ item.name }}
                </span>
                <i class="close" :itemId="item.id">×</i>
            </li>
        </ul>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                items: [], //项目集合
            }
        },
        mounted() {
            this.updateItemsSort();
            //删除元素事件
            document.querySelectorAll(".list .close").forEach(c => {
                c.addEventListener("click", () => {
                    let id = c.getAttribute("itemId");
                    for (let i = 0; i < this.items.length; i++) {
                        if (this.items[i].id == id) {
                            this.items.splice(i, 1);
                            this.updateSort();
                            break;
                        }
                    }
                })
            })

            //拖拽元素集合
            let $dragList = document.querySelectorAll(".list span");
            let dragFrom = -1;
            let dragTo = -1;

            $dragList.forEach(v => {
                //拖动元素开始拖拽
                v.addEventListener('dragstart', (ev) => {
                    console.log("item" + ev.target.getAttribute("sort") + "_dragstart");
                    ev.dataTransfer.effectAllowed = "move";
                    //获取拖动元素排序
                    dragFrom = parseInt(ev.target.getAttribute("sort"));
                    //添加拖动元素当前样色
                    ev.target.classList.add("dragFrom");
                });

                //当拖动元素进入可释放目标时
                v.addEventListener('dragenter', (ev) => {
                    console.log("item" + ev.target.getAttribute("sort") + "_dragenter");
                    let _dragTo = ev.target.getAttribute("sort");

                    //添加目标元素当前样色
                    $dragList.forEach(item => {
                        item.classList.remove("dragTo");
                    });
                    if ( dragFrom != _dragTo){
                        ev.target.classList.add("dragTo");
                    }
                    ev.preventDefault();
                });

                //当拖动元素在可释放目标上时
                v.addEventListener('dragover', (ev) => {
                    console.log("item" + ev.target.getAttribute("sort") + "_dragover");
                    ev.preventDefault();
                });

                //当拖动元素在可释放目标上被释放时
                v.addEventListener('drop', (ev) => {
                    console.log("item" + ev.target.getAttribute("sort") + "_drop");
                    ev.preventDefault();
                    //获取目标元素id
                    dragTo = parseInt(ev.target.getAttribute("sort"));

                    if (dragFrom == dragTo) return;
                    console.log("%citem" + dragFrom + "交换item" + dragTo, "color:orange");

                    //交换排序
                    let temp = this.items[dragFrom].sort;
                    this.items[dragFrom].sort = this.items[dragTo].sort;
                    this.items[dragTo].sort = temp;

                    ev.target.classList.remove("dragTo");
                    this.updateItemsSort();

                });

                //拖动元素结束拖拽
                v.addEventListener('dragend', (ev) => {
                    console.log("item" + ev.target.getAttribute("sort") + "_dragend");
                    ev.target.classList.remove("dragFrom");
                    ev.preventDefault();
                });

                //当拖动元素变得不再是拖动操作的选中目标时
                v.addEventListener('dragexit', (ev) => {
                    console.log("item" + ev.target.getAttribute("sort") + "_dragexit");
                    ev.preventDefault();
                });

                //当拖动元素离开一个可释放目标时触发
                v.addEventListener('dragleave', (ev) => {
                    console.log("item" + ev.target.getAttribute("sort") + "_dragleave");
                    ev.preventDefault();
                });


            })
        },
        updated() {
            //当项目集合变更后，触发li元素变更，再执行li的位置变更
            this.updateItemsSort();
        },
        methods: {
            //通过translate改变排序位置
            updateItemsSort() {
                let $list = document.querySelectorAll(".list li");
                let _items = [];
                this.items.forEach((item, i) => {
                    let y = parseInt(item.sort / 3);
                    let x = item.sort % 3;
                    $list[i].setAttribute("style", "transform: translate( " + 100 * x + "%, " + 100 * y +
                        "%)");
                    _items.push({ id: item.id, sort: item.sort });
                });

                //存储排序
                if (!!window.localStorage) {
                    window.localStorage["listSort"] = JSON.stringify(_items);
                }
            },
            //更新排序
            updateSort() {
                //已有排序的先按现有排序排列
                this.items.sort((a, b) => {
                    return a.sort - b.sort
                });
                //再重新设置排序编号，以防元素个数发生变更
                this.items.forEach((item, index) => {
                    item.sort = index;
                });
            }
        },

        created() {
            this.items = [
                { id: 0, name: "item0" },
                { id: 1, name: "item1" },
                { id: 2, name: "item2" },
                { id: 3, name: "item3" },
                { id: 4, name: "item4" },
                { id: 5, name: "item5" },
                { id: 6, name: "item6" },
                { id: 7, name: "item7" },
                { id: 8, name: "item8" },
            ];

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
        }
    }
</script>

<style>
    * {
        margin: 0;
        padding: 0;
    }

    h1 {
        height: 50px;
        line-height: 50px;
        font-size: 20px;
        text-align: center;
    }

    html,
    body,
    #app {
        height: 100%;
    }

    .list,
    .list li {
        list-style: none;
    }

    .list {
        position: absolute;
        left: 10px;
        right: 10px;
        top: 50px;
        bottom: 10px;
        overflow: hidden;

    }

    .list li {
        position: absolute;
        width: 33.33%;
        height: 33.33%;
        transition: all .4s;
    }

    .list span {
        position: absolute;
        left: 10px;
        top: 10px;
        right: 10px;
        bottom: 10px;
        line-height: 2;
        display: block;
        background: #000;
        color: #fff;
        font-size: 20px;
        text-align: center;
        cursor: move;
        transition: all .4s;
    }

    .list .close {
        position: absolute;
        right: 15px;
        top: 15px;
        width: 20px;
        height: 20px;
        border-radius: 20px;
        line-height: 17px;
        text-align: center;
        background: rgba(255, 255, 255, .5);
        font-style: normal;
        cursor: pointer;
    }

   .list .dragFrom {
       background: green;
    }
   .list .dragTo {
       background: green;
    }
</style>
