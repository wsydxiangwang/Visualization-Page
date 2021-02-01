<template>
    <section class="decoration-edit">
        <section class="l">
            <ul 
                @dragstart="dragStart"
                @dragend="dragEnd"
            >
                <li 
                    v-for="(val, key, index) in typeList"
                    draggable 
                    :data-type="key"
                    :key="index + 1"
                >
                    <span :class="val.icon"></span>
                    <p>{{val.name}}</p>
                </li>
            </ul>
        </section>
        <section class="c">
            <!-- header 不可拖拽 --> 
            <div class="top-nav" @click="selectType(0)">
                <img src="./../assets/images/topNavBlack.png">
                <span class="tit">{{ info.title }}</span>
            </div>
            <div 
                class="view-content"
                @drop="drog"
                @dragover="dragOver"
                :style="{ backgroundColor: info.backgroundColor }"
            >
                <Draggable
                    v-model="view"
                    draggable=".item"
                >
                    <template v-for="(item, index) in view">
                        <div
                            v-if="index > 0"
                            :data-index="index"
                            :key="index"
                            class="item"
                            @click="selectType(index)"
                        >
                            <!-- waiting -->
                            <template v-if="item.status && item.status == 2">
                                <div class="wait"> {{ item.type }} </div>
                            </template>
                            <template v-else>
                                <component 
                                    :is="typeList[item.type]['com']" 
                                    :data="item"
                                    :className="className[item.tabType]"
                                ></component>
                            </template>
                            <i @click="deleteItem($event, index)" class="el-icon-error"></i>
                        </div>
                    </template>
                </Draggable>
            </div>
        </section>
        <section class="r">
            <EditForm
                :data="props"
                v-if="isRight"
            ></EditForm>
        </section>

        <div class="submit-btn">
            <el-button @click="submit">提交页面</el-button>
        </div>
    </section>
</template>

<script>
import Draggable from 'vuedraggable'
// import { imageUpload } from '@/api/commodity'
import EditForm from '@/components/Edit/index'
import Product from '@/components/View/Product'
import Images from '@/components/View/Images'
import Banner from '@/components/View/Banner'
export default {
    components: {
        EditForm,
        Draggable,
        Product,
        Images,
        Banner
    },
    data() {
        return {
            typeList: {
                banner: {
                    name: '轮播图',
                    icon: 'el-icon-picture',
                    com: Banner
                },
                product: {
                    name: '商品',
                    icon: 'el-icon-s-goods',
                    com: Product
                },
                images: {
                    name: '图片',
                    icon: 'el-icon-picture',
                    com: Images
                },
            }, 
            view: [
                {
                    type: 'info',
                    title: '页面标题'
                }
            ],
            title: '页面标题',
            type: '',
            index: null,        // 当前组件的索引
            isPush: false,      // 是否已添加组件

            props: {},          // 传值
            isRight: false,

            className: {
                1: 'one',
                2: 'two',
                3: 'three'
            }
        }
    },
    computed: {
        info() {
            return this.view[0]
        }
    },
    methods: {
        submit() {
            // JSON 转换会丢失 formData
            const form = JSON.parse(JSON.stringify(this.view))

            if (form.length == 1) {
                this.$message.error('请添加模块！')
                return
            }

            for (let i of form) {
                if (i.data && i.data.length == 0) {
                    this.$message.error('请填写完整信息！')
                    return
                }
                if (i.type === 'product') {
                    i.data = i.data.map(val => val.productId).join(',')
                }
            }

            this.$message.success('数据提交成功，请按F12打开控制台查看待提交数据集合！')

            console.log(form)  // 提交的数据，根据接口形式修改

            return

            // 上传图片，修改数据字段（兼容接口
            const uploadPromise = {}
            for (let i = 0; i < this.view.length; i++) {
                if (['images', 'banner'].includes(this.view[i].type)) {
                    uploadPromise[i] = []
                    let list = this.view[i]['data']
                    list.forEach((item, index) => {
                        uploadPromise[i].push(
                            new Promise((resolve, reject) => {
                                const link = item.link
                                const name = item.name
                                imageUpload(item.form).then(res => {    // 上传图片接口
                                    form[i]['data'][index] = {
                                        url: res.data.url,
                                        link,
                                        name
                                    }
                                    resolve(res)
                                }).catch(err => {
                                    reject(err)
                                })
                            })
                        )
                    })
                }
            }

            let [len, count] = [Object.keys(uploadPromise).length, 0]
            if (len) {
                for (let i in uploadPromise) {
                    Promise.all(uploadPromise[i]).then(res => {
                        count++
                        if (count === len) {
                            // 提交数据
                        }
                    }).catch(err => {
                        this.$message.error(err)
                    })
                }
            } else {
                // 提交数据
            }
        },
        // 切换视图组件
        selectType(index) {
            this.isRight = false
            this.props = this.view[index]
            this.$nextTick(() => this.isRight = true)
        },
        deleteItem(e, index) {
            e.preventDefault()
            e.stopPropagation()
            this.view.splice(index, 1)
            this.isRight = false
            this.props = {}
        },
        // 拖拽类型
        dragStart(e) {
            this.type = e.target.dataset.type
        },
        // 结束拖拽
        dragEnd(e) {
            this.$delete(this.view[this.index], 'status')
            this.isPush = false
            this.type = null
        },
        // 已放置到指定位置
        drog(e) {
            if (!this.type) { // 内容拖拽
                return
            }
            e.preventDefault()
            e.stopPropagation()
            this.dragEnd()
        },
        // 移动中
        dragOver(e) {
            if (!this.type) { // 内容拖拽
                return
            }
            e.preventDefault()
            e.stopPropagation()

            let className = e.target.className
            let name = className !== 'view-content' ? 'item' : 'view-content'
            
            const defaultData = {
                type: this.type,    // 组件类型
                status: 2,          // 默认状态
                data: [],           // 数据
                options: {}         // 选项操作
            }

            if (name == 'view-content') {
                if (!this.isPush) {
                    this.index = this.view.length
                    this.isPush = true
                    this.view.push(defaultData)
                }
            } else if (name == 'item') {
                
                let target = e.target
                let [ y, h, curIndex ] = [ e.offsetY, target.offsetHeight, target.dataset.index ]
                let direction = y < (h / 2)

                if (!this.isPush) {
                    // Push to Top or Bottom
                    if (direction) {
                        if (curIndex == 0) {
                            this.view.unshift(defaultData)
                        } else {
                            this.view.splice(curIndex, 0, defaultData)
                        }
                    } else {
                        curIndex = +curIndex + 1
                        this.view.splice(curIndex, 0, defaultData)
                    }
                } else {
                    // Moving
                    if (direction) {
                        var i = curIndex == 0 ? 0 : curIndex - 1
                        var result = this.view[i]['status'] == 2
                    } else {
                        var i = +curIndex + 1
                        var result = this.view.length > i && this.view[i]['status'] == 2
                    }

                    if (result) return

                    const temp = this.view.splice(this.index, 1)
                    this.view.splice(curIndex, 0, temp[0])
                }
                this.index = curIndex
                this.isPush = true
            }
        }
    }
}
</script>
<style lang="scss">
.decoration-edit{
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px 0;
    background: #f7f8f9;
    height: calc(100vh - 50px);
    position: relative;
    .l, .r {
        width: 340px;
        height: 100%;
        padding: 15px 0;
        background: #fff;
    }
    .l{
        ul{
            margin: 0;
            padding: 0;
            li{
                width: 80px;
                height: 80px;
                display: flex;
                justify-content: center;
                align-items: center;
                flex-direction: column;
                cursor: default;
                list-style: none;
                font-size: 14px;
                color: #666;
                float: left;
                margin: 0 10px;
                border-radius: 6px;
                transition: all .3s;
                cursor: pointer;
                &:hover{
                    background: #efefef;
                }
                span{
                    display: block;
                    font-size: 40px;
                    margin-bottom: 8px;
                    color: #999;
                }
                p{
                    display: block;
                    margin: 0;
                    font-size: 12px;
                }
            }
        } 
    }
    .c{
        width: auto;
        max-width: 400px;
        position: relative;
        .top-nav{
            position: absolute;
            top: 0;
            background: #fff;
            z-index: 999;
            transition: all .3s;
            & * {
                pointer-events: none;
            }
            &:hover{
                transform: scale(0.95);
                border-radius: 10px;
                overflow: hidden;
                box-shadow: 0 0 10px #afafaf;
            }
            .tit{
                position: absolute;
                left: 50%;
                bottom: 10px;
                transform: translateX(-50%);
            }
            img{
                max-width: 100%;
                image-rendering: -moz-crisp-edges;
                image-rendering: -o-crisp-edges;
                image-rendering: -webkit-optimize-contrast;
                image-rendering: crisp-edges;
                -ms-interpolation-mode: nearest-neighbor;
            }
        }
        .view-content{
            width: 400px;
            height: 700px;
            background: #f5f5f5;
            overflow-y: auto;
            overflow-x: hidden;
            padding-top: 72px;
            box-shadow: 0 2px 6px #ccc;
            &::-webkit-scrollbar{
                width: 6px;
            }
            &::-webkit-scrollbar-thumb{
                background: #dbdbdb;
            }
            &::-webkit-scrollbar-track{
                background: #f6f6f6;
            }
            
            .item{
                transition: all .3s;
                background: #fff;
                &:hover{
                    transform: scale(0.95);
                    border-radius: 10px;
                    box-shadow: 0 0 10px #afafaf;
                    .el-icon-error{
                        display: block;
                    }
                }
                div{
                    pointer-events: none;
                }
                .wait{
                    background: #deedff;
                    height: 35px;
                    text-align: center;
                    line-height: 35px;
                    font-size: 12px;
                    color: #666;
                }
                .el-icon-error{
                    position: absolute;
                    right: -10px;
                    top: -6px;
                    color: red;
                    font-size: 25px;
                    cursor: pointer;
                    display: none;
                    z-index: 9999;
                }
            }
        }
    }    
    .submit-btn{
        position: absolute;
        bottom: 30px;
        left: 50%;
        transform: translateX(-50%);
    }
}
</style>