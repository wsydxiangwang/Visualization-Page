<template>
    <div class="banner">
        <el-carousel 
            height="176px" 
            v-if="data.data && data.data.length > 0" 
        >
            <el-carousel-item v-for="(item, index) in data.data" :key="index">
                <img :src="item.url">
            </el-carousel-item>
        </el-carousel>
        <div class="image-null" v-else><span class="el-icon-picture"></span></div>
    </div>
</template>

<script>
export default {
    name: 'Banner',
    props: ['data']
}
</script>

<style lang="scss" scoped>
.banner{
    font-size: 0;
    img{
        width: 100%;
    }
    .el-carousel__indicator--horizontal{
        padding: 12px 4px;
    }
    .el-carousel__button{
        width: 12px;
    }
    .image-null{
        background: #ffffff;
        height: 176px;
        font-size: 50px;
        display: flex;
        align-items: center;
        justify-content: center;
        color: #c1c1c1;
    }
}
</style>