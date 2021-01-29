<template>
    <!-- Banner & Image 通用组件 -->
    <div class="image-content">
        <p class="desc">添加图片 (最多{{ len }}张，可拖动排序）</p>
        <vuedraggable
            v-model="list.data"
            tag="ul"
            draggable="li"
            v-if="list.data && list.data.length > 0" 
            class="image-list"
            :class="{ disable: data.tabType == 2 }"
        >
            <li v-for="(item, index) in list.data" :key="index">
                <div class="l-info">
                    <p><span class="sort">排序{{ index + 1 }}</span></p>
                    <p>
                        <span>名称：</span>
                        <span class="text">{{ item && item.name }}</span>
                    </p>
                    <p>
                        <span>链接：</span>
                        <el-tooltip effect="dark" :content="item.link" placement="top" v-if="item.link">
                            <span class="text" @click="urlPopup(index, item.link)">{{ item.link }}</span>
                        </el-tooltip>
                        <span v-else @click="urlPopup(index)" class="link">请输入跳转链接</span>
                    </p>
                </div>
                <div class="r-image">
                    <span @click="removeImage(index)" class="el-icon-close"></span>
                    <div class="image-box">
                        <img :src="item && item.url">
                        <span @click="addImage(index)" class="el-icon-edit-outline"></span>
                    </div>
                </div>
            </li>
        </vuedraggable>
        <template v-if="list.data && list.data.length < len" >
            <el-button 
                type="primary" 
                icon="el-icon-plus" 
                @click="addImage(null)"
                class="add-image"
            >
                添加图片
            </el-button>
            <p class="size">（建议尺寸：{{ size }}）</p>
        </template>
        <el-upload
            ref="upload"
            :http-request="upload"
            :show-file-list="false"
            multiple
            action
            style="display:none"
        >
        </el-upload>
        <el-dialog title="请填写图片跳转链接" :visible.sync="show" @close="close">
            <el-form label-width="100px">
                <el-form-item label="跳转链接">
                    <el-input v-model="url"></el-input>
                </el-form-item>
                <el-form-item label="">
                    <el-button type="primary" @click="confirm">确定</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>
    </div>
</template>

<script>
import vuedraggable from 'vuedraggable'
export default {
    name: 'Images',
    components: {
        vuedraggable
    },
    props: {
        data: {
            type: Object,
            default: () => {}
        }
    },
    data() {
        return {
            list: {},
            url: '',
            index: null,
            show: false,
            imageIndex: null,
        }
    },
    computed: {
        size() {
            return this.list.type == 'images' ? '750*750' : '750*400'
        },
        len() {
            return this.list.type == 'images' ? 8 : 10
        }
    },
    mounted() {
        this.list = this.data
    },
    methods: {
        close() {
            this.show = false
            this.url = ''
        },
        confirm() {
            this.list['data'][this.index]['link'] = this.url
            this.close()
        },
        urlPopup(index) {
            this.show = true
            this.index = index
            this.url = link
        },
		removeImage(index) {
            this.list.data.splice(index, 1)
		},
        addImage(index) {
			this.imageIndex = index
			this.$refs['upload'].$children[0].$refs.input.click()
        },
        upload(params) {
            const len = this.list.data.length;
            
            if (len >= this.len) {
                this.$message.error(`最多添加${this.len}张图片!`);
                return
            }
			const file = params.file,
				  fileType = file.type;

            const verifyList = [
                    {
                        text: "只能上传图片格式png、jpg、gif!",
                        result: fileType.indexOf('image') != -1
                    },
                    {
                        text: "只能上传大小小于5M",
                        result: file.size / 1024 / 1024 < 5
                    }
                ]

            for (let item of verifyList) {
                if (!item.result) {
                    this.$message.error(item.text)
                    return
                }
            }

			const form = new FormData();
			form.append("file", file);
			form.append("clientType", "multipart/form-data");

            const index = this.imageIndex
            const data = { 
                name: file.name, 
                url: URL.createObjectURL(file), 
                form
            }
            
            if (index !== null) {
                this.$set(this.list.data, index, data)
            } else {
                this.list.data.push(data)
            }
		}
    }
}
</script>

<style lang="scss" scoped>
.image-content{
    display: flex;
    flex-direction: column;
    flex-wrap: nowrap;
    height: 100%;
    overflow: hidden;
    .desc{
        text-align: center;
        font-size: 12px;
        color: #666;
        margin: 18px 0;
        padding-bottom: 10px;
        border-bottom: 1px dashed #ddd;
    }
    .size{
        text-align: center;
        font-size: 12px;
        color: #999;
        margin-bottom: 0;
    }
    .add-image{
        width: calc(100% - 20px);
        height: 34px;
        line-height: 34px;
        padding: 0;
        font-size: 12px;
        margin-left: 10px;
        margin-top: 10px;
    }
    .image-list{
        margin: 0;
        padding: 0 10px;
        overflow: auto;
        &::-webkit-scrollbar-thumb{
            background: #dbdbdb;
            border-radius: 10px;
        }
        &::-webkit-scrollbar-track{
            background: #f6f6f6;
            border-radius: 10px;
        }
        &::-webkit-scrollbar{
            width: 6px;
        }
        li{
            list-style: none;
            background: #f7f8f9;
            border-radius: 4px;
            padding: 6px 14px 10px;
            margin-bottom: 10px;
            display: flex;
            justify-content: space-between;
            &.disable div{
                pointer-events: none;
                user-select: none;
            }
            .l-info{
                font-size: 12px;
                padding-top: 8px;
                width: calc(100% - 70px);
                p{
                    margin: 12px 0 0;
                    white-space: nowrap;
                    overflow: hidden;
                    display: flex;
                    .link{
                        color: #1b8bff;
                        cursor: pointer;
                    }
                    .text{
                        white-space: nowrap;
                        text-align: -webkit-auto;
                        text-overflow: ellipsis;
                        overflow: hidden;
                    }
                }
            }
            .r-image{
                text-align: right;
                .el-icon-close{
                    color: #999;
                    font-size: 12px;
                    font-weight: 600;
                    margin-bottom: 6px;
                    cursor: pointer;
                    &:hover{
                        color: red;
                    }
                }
                .image-box{
                    width: 70px;
                    height: 70px;
                    border-radius: 5px;
                    overflow: hidden;
                    position: relative;
                    background: #fff;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    &::before{
                        content: '';
                        position: absolute;
                        top: 0;
                        left: 0;
                        width: 100%;
                        height: 100%;
                        background: rgba(0,0,0,.5);
                        opacity: 0;
                        transition: all .3s;
                    }
                    span{
                        position: absolute;
                        top: 50%;
                        left: 50%;
                        color: #fff;
                        transform: translate(-50%, -50%);
                        font-size: 20px;
                        cursor: pointer;
                        opacity: 0;
                        transition: all .3s;
                    }
                    img{
                        max-width: 100%;
                    }
                    &:hover{
                        &::before, span{
                            opacity: 1;
                        }
                    }
                }
            }
        }
    }
}
</style>