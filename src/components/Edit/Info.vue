<template>
    <div class="info-content">
        <el-form label-width="80px">
            <el-form-item label="页面标题">
                <el-input v-model="list.title"></el-input>
            </el-form-item>
            <el-form-item label="页面备注">
                <el-input type="textarea" :rows="4" v-model="list.remarks"></el-input>
            </el-form-item>
            <el-form-item label="背景颜色">
                <el-color-picker v-model="list.backgroundColor" show-alpha></el-color-picker>
            </el-form-item>
        </el-form>
    </div>
</template>

<script>
export default {
    name: 'Info',
    props: ['data', 'className'],
    data() {
        return {
            list: {}
        }
    },
    mounted() {
        this.list = this.data
    }
}
</script>

<style lang="scss" scoped>
.info-content{
    padding: 20px;
}
</style>