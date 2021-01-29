<template>
    <div 
        class="image-box" 
        :class="className"
    >
        <template v-if="data.data && data.data.length > 0">
            <a 
                v-for="(item, index) in data.data"
                :key="index"
                :href="item.link"
            >
                <img :src="item.url">
            </a>
        </template>
        <template v-else>
            <div class="image-null"><span class="el-icon-picture"></span></div>
        </template>
    </div>
</template>

<script>
export default {
    name: 'Images',
    props: ['data', 'className']
}
</script>

<style lang="scss" scoped>
.image-box{
    font-size: 0;
    img{
        width: 100%;
        max-width: 100%;
    }
    .image-null{
        background: #ffffff;
        height: 300px;
        font-size: 50px;
        display: flex;
        align-items: center;
        justify-content: center;
        color: #c1c1c1;
        width: 100%;
    }
    a{
        display: block;
    }
    &.two{
        display: flex;
        flex-wrap: wrap;
        box-sizing: content-box;
        a{
            width: 50%;
            box-sizing: border-box;
            position: relative;
            overflow: hidden;
            display: block;
            padding-bottom: 50%;
            background: #e4e4e4;
            img{
                position: absolute;
            }
        }
    }
    &.three{
        height: 100px;
        overflow-y: auto;
        white-space: nowrap;
        box-sizing: content-box;
        a{
            width: 100px;
            height: 100px;
            margin: 0 4px;
            display: inline-block;
            vertical-align: bottom;
            background: #e4e4e4;
        }
    }
}
</style>