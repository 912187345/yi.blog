<template>
    <el-card class="box-card settingWrap">
        <div class="line">
            <div class="title">头像：</div>
            <el-upload
                class="upload-demo"
                drag
                action="/usersetting/upload-head-image"
                :file-list="fileList"
                :data="uploadParams"
                :on-success="uploadHeadSuccess"
                :on-error="uploadError"
                :before-upload="beforeUploadHead"
                :limit="1">
                <i class="el-icon-upload"></i>
                <div class="el-upload__text">将图片拖到此处，或<em>点击上传</em></div>
                <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过1MB</div>
            </el-upload>
        </div>
        <div class="line">
            <div class="title">背景：</div>
            <el-upload
                class="upload-demo"
                drag
                action="/usersetting/set-background"
                :file-list="fileListGB"
                :data="uploadParams"
                :on-success="uploadBGSuccess"
                :on-error="uploadBGError"
                :before-upload="beforeUploadBg"
                :limit="1">
                <i class="el-icon-upload"></i>
                <div class="el-upload__text">将图片拖到此处，或<em>点击上传</em></div>
                <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过2MB</div>
            </el-upload>
        </div>
        <div class="line" @click="edit('username')">
            <div class="title">用户名：</div>
            <div>{{ userInfo.username }}</div>
            <div class="btn">修改</div>
        </div>
        <div class="line" @click="edit('email')">
            <div class="title">邮箱地址：</div>
            <div>{{  userInfo.email }}</div>
            <div class="btn">修改</div>
        </div>
    </el-card>
</template>

<script>
import {mapState,mapActions} from 'vuex'
 export default {
    data() {
      return {
        fileList: [],
        fileListGB: []
      };
    },
    computed:{
        ...mapState(['userToken','userInfo']),
        uploadParams(){
            return {
                token:this.userToken
            }
        }
    },
    methods: {
        ...mapActions({editUser:'editUser'}),
        uploadHeadSuccess(res,file){
            if( res.status === 'success' ){
                this.$store.commit('setUserHead',res.data.headImg);
                this.$message({
                    message: '上传头像成功',
                    type: 'success'
                });
            }else{
                this.$message('上传失败，请稍后重试');
            }
        },
        uploadBGSuccess(res,file){
            if( res.status === 'success' ){
                this.$store.commit('setBackground',res.data.background);
                this.$message({
                    message: '上传背景成功',
                    type: 'success'
                });
            }else{
                this.$message('上传失败，请稍后重试');
            }
        },
        uploadError(err){
            this.$message('上传失败，请稍后重试');
        },
        uploadBGError(err){
            this.$message('上传失败，请稍后重试');
        },
        edit(type){
            let tips,errMes,rex;
            switch(type){
                case 'username':
                    tips = '请输入新的用户名';
                    errMes = '请输入内容';
                    rex = /\S/;
                    break;
                case 'email':
                    tips = '请输入新的邮箱地址';
                    errMes = '请输入正确的邮箱';
                    rex = /[\w!#$%&'*+/=?^_`{|}~-]+(?:\.[\w!#$%&'*+/=?^_`{|}~-]+)*@(?:[\w](?:[\w-]*[\w])?\.)+[\w](?:[\w-]*[\w])?/;
                    break;
            }
            this.$prompt(tips, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    inputPattern: rex,
                    inputErrorMessage: errMes
                }).then(({ value }) => {
                    let params = {
                            type:type,
                            value:value
                        }
                    this.editUser(params)
                    .then((rst)=>{
                        if( rst.status == 'success' ){
                            let userInfo = this.userInfo;
                            switch(type){
                                case 'username':
                                    userInfo.username = value;
                                    break;
                                case 'email':
                                    userInfo.email = value;
                                    break;
                            }
                            this.$store.commit('setUserInfo',userInfo);
                            this.$message({
                                type: 'success',
                                message: '修改成功'
                            });  
                        }
                    })
                },err=>{
                    this.$message({
                        type: 'info',
                        message: '取消修改'
                    });      
                })
                .catch(() => {
                    this.$message({
                        type: 'warning',
                        message: '修改失败，请稍后重试'
                    });
            });
        },
        beforeUploadHead(file){
            const isJPG = file.type === 'image/jpeg';
            const isLtS = file.size / 1024 / 1024 < 1;
            if (!isJPG) {
                this.$message.error('上传头像图片只能是 JPG 格式!');
            }
            if (!isLtS) {
                this.$message.error('上传头像图片大小不能超过 1MB!');
            }
            return isJPG && isLtS
        },
        beforeUploadBg(file){
            const isJPG = file.type === 'image/jpeg';
            const isLtS = file.size / 1024 / 1024 < 2;
            if (!isJPG) {
                this.$message.error('上传背景图片只能是 JPG 格式!');
            }
            if (!isLtS) {
                this.$message.error('上传背景图片大小不能超过 2MB!');
            }
            return isJPG && isLtS
        }
    }
  }
</script>

<style lang='scss'>
.avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
}
.avatar-uploader .el-upload:hover {
    border-color: #409EFF;
}
.avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
}
.avatar {
    width: 178px;
    height: 178px;
    display: block;
}
.settingWrap{
    @include blogList;
    .line{
        display: flex;
        margin: 20px 0;
    }
    .title{
        width: 90px;
    }
    .btn{
        margin-left: 20px;
        cursor: pointer;
        &:hover{
            color: skyblue;
        }
    }
}
</style>
