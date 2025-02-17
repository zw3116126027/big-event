<template>
    <el-card class="page-container">
        <template #header>
            <div class="header">
                <span>基本资料</span>
            </div>
        </template>
        <el-row>
            <el-col :span="12">
                <el-form :model="registerData" :rules="rules" label-width="100px" size="large">
                    <el-form-item>
                        <h1>重置密码</h1>
                    </el-form-item>
                    <el-form-item prop="old_pwd" label="旧密码">
                        <el-input type="password" v-model="registerData.old_pwd"></el-input>
                    </el-form-item>
                    <el-form-item prop="new_pwd" label="新密码">
                        <el-input type="password" v-model="registerData.new_pwd"></el-input>
                    </el-form-item>
                    <el-form-item prop="re_pwd" label="确认密码">
                        <el-input type="password" v-model="registerData.re_pwd"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="updatePasswordInfo">提交</el-button>
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
    </el-card>
</template>

<script setup>
    //修改密码
    import { ref } from 'vue';
    import useUserInfoStore from '@/stores/userInfo.js';
    import { updatePasswordService } from '@/api/user.js';
    import { ElMessage } from 'element-plus';
    import { useRouter } from 'vue-router'
    import { useTokenStore } from '@/stores/token.js'
    const tokenStore = useTokenStore()
    const router = useRouter()


    const userInfoStore = useUserInfoStore();
    //用户信息
    const userInfo = ref({ ...userInfoStore.info });

    const registerData = ref({
        old_pwd: "",
        new_pwd: "",
        re_pwd: ""
    });
    //重置密码
    const updatePasswordInfo = async () => {

        let result = await updatePasswordService(registerData.value)
        ElMessage.success(result.message ? result.message : "修改成功")
        //退出登录
        //1.清空pinia中存储的token以及个人信息
        tokenStore.removeToken()
        userInfoStore.removeInfo()
        router.push('/login')
        ElMessage({

            type: 'success',
            message: '正在退出登录',
        })
    }

    const checkRepassword = (rule, value, callback) => {
        if (value === '') {
            callback(new Error('请再次确认密码'));
        } else if (value !== registerData.value.new_pwd) {
            callback(new Error("请确保两次输入的密码一样"));
        } else {
            callback();
        }
    };


    const rules = {
        old_pwd: [
            { required: true, message: "请输入旧密码", trigger: "blur" },
            { min: 5, max: 16, message: "长度为5~16位非空字符", trigger: "blur" }
        ],
        new_pwd: [
            { required: true, message: "请输入新密码", trigger: "blur" },
            { min: 5, max: 16, message: "长度为5~16位非空字符", trigger: "blur" }
        ],
        re_pwd: [
            { required: true, message: "请确认新密码", trigger: "blur" },
            { validator: checkRepassword, trigger: "blur" }
        ]
    };

    const updateUserInfo = async () => {
        const tempUserInfo = { ...userInfo.value };
        let result = await userInfoUpdateService(tempUserInfo);

        ElMessage.success(result.message ? result.message : "修改成功");

        userInfoStore.setInfo(tempUserInfo);
    };
</script>