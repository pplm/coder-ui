<style lang="less">
    @import './login.less';
</style>

<template>
    <div class="login" @keydown.enter="handleSubmit">
        <div class="login-con">
            <Card :bordered="false">
                <p slot="title">
                    <Icon type="log-in"></Icon>
                    欢迎登录
                </p>
                <div class="form-con">
                    <Form ref="loginForm" :model="form" :rules="rules">
                        <FormItem prop="userName">
                            <Input v-model="form.userName" placeholder="请输入用户名">
                                <span slot="prepend">
                                    <Icon :size="16" type="person"></Icon>
                                </span>
                            </Input>
                        </FormItem>
                        <FormItem prop="password">
                            <Input type="password" v-model="form.password" placeholder="请输入密码">
                                <span slot="prepend">
                                    <Icon :size="14" type="locked"></Icon>
                                </span>
                            </Input>
                        </FormItem>
                        <FormItem>
                            <Button @click="handleSubmit" type="primary" long>登录</Button>
                        </FormItem>
                    </Form>
                    <p class="login-tip">输入任意用户名和密码即可</p>
                </div>
            </Card>
        </div>
    </div>
</template>

<script>
import Cookies from 'js-cookie';
import util from '@/libs/util.js';

export default {
    data () {
        return {
            form: {
                userName: 'iview_admin',
                password: ''
            },
            rules: {
                userName: [
                    { required: true, message: '账号不能为空', trigger: 'blur' }
                ],
                password: [
                    { required: true, message: '密码不能为空', trigger: 'blur' }
                ]
            }
        };
    },
    methods: {
        handleSubmit () {
            let _self = this
            this.$refs.loginForm.validate((valid) => {
                if (valid) {
                    util.ajax.post('/system/login', {
                        username: this.form.userName,
                        password: require('node-sha1')(this.form.password)
                    }).then(res => {
                        console.log(res)
                        if (res.status === 200) {
                            if (res.data.code === '0') {
                                Cookies.set('user', this.form.userName)
                                Cookies.set('token', res.data.content.token)
                                _self.$store.commit('setAvator', 'https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=3448484253,3685836170&fm=27&gp=0.jpg')
                                if (_self.form.userName === 'iview_admin') {
                                    Cookies.set('access', 0)
                                } else {
                                    Cookies.set('access', 1)
                                }
                                _self.$router.push({
                                    name: 'home_index'
                                })
                            } else {
                                this.$Message.error('用户名密码错误！')
                            }
                        }
                    }).catch (err => {
                        console.log(err)
                    })
                }
            });
        }
    }
};
</script>

<style>

</style>
