<template>
    <el-container class="root-container">
        <el-dialog
                title="表情包标引工具"
                class="login-dialog"
                :visible.sync="loginVisible"
                :fullscreen="true"
                :close-on-click-modal="false"
                :close-on-press-escape="false"
                :show-close="false"
                center>
            <el-form>
                <el-form-item label="你的名字">
                    <el-input v-model="config.admin"></el-input>
                </el-form-item>
                <el-form-item label="口令">
                    <el-input v-model="password"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button type="primary" @click="login">登 录</el-button>
            </span>
        </el-dialog>
        <el-header>
            <el-row gutter="10px" class="header-title">
                <el-col span="14" class="big-title">表情包标引 v1</el-col>
                <el-col span="5" class="admin-name el-icon-user-solid">
                    当前登录人：{{config.admin}}
                </el-col>
                <el-col span="5">
                    <el-button icon="el-icon-edit" size="small" class="info-button" show-close="false"
                               @click="helpInfoVisible = true">说明
                    </el-button>
                    <el-button type="primary" plain icon="el-icon-setting" size="small" class="info-button"
                               @click="settingVisible = true">设置
                    </el-button>
                </el-col>
                <el-dialog class="config-dialog" title="选择标引项目" :visible.sync="settingVisible" width="500px"
                           show-close="false">
                    <el-form>
                        <el-row>
                            <el-col span="8">
                                <el-form-item label="文字类型">
                                    <el-switch v-model="config.visible.textType"></el-switch>
                                </el-form-item>
                            </el-col>
                            <el-col span="8">
                                <el-form-item label="角色">
                                    <el-switch v-model="config.visible.role"></el-switch>
                                </el-form-item>
                            </el-col>
                            <el-col span="8">
                                <el-form-item label="情绪">
                                    <el-switch v-model="config.visible.emotion"></el-switch>
                                </el-form-item>
                            </el-col>
                        </el-row>
                        <el-row>
                            <el-col span="8">
                                <el-form-item label="风格">
                                    <el-switch v-model="config.visible.style"></el-switch>
                                </el-form-item>
                            </el-col>
                            <el-col span="8">
                                <el-form-item label="主题">
                                    <el-switch v-model="config.visible.topic"></el-switch>
                                </el-form-item>
                            </el-col>
                            <el-col span="8">
                                <el-form-item label="描述">
                                    <el-switch v-model="config.visible.description"></el-switch>
                                </el-form-item>
                            </el-col>
                        </el-row>
                    </el-form>
                    <div slot="footer" class="dialog-footer">
                        <el-button type="primary" @click="settingVisible = false">确 定</el-button>
                    </div>
                </el-dialog>
                <el-drawer
                        title="帮助说明"
                        :visible.sync="helpInfoVisible"
                        :with-header="false">
                    <div class="intro">
                        <h3>使用说明：</h3>
                        <p class="sub-intro">系统很垃圾，请小心使用，可能数据说没就没了。但是也不用太担心，最多少掉当前正在操作的数据。</p>
                        <p class="sub-intro">多使用<span>"从数据库同步"</span>按钮，可以确认到底有没有操作成功。</p>
                        <h4>快捷键说明：</h4>
                        <p class="sub-intro">所有快捷键均当<span>鼠标焦点在表单中</span>的时候才能使用</p>
                        <p>提交当前修改并进入下一个：ctrl + enter</p>
                        <p>新增标签：enter</p>
                        <p>上一个：ctrl + [</p>
                        <p>下一个：ctrl + ]</p>
                        <p>选择文字类型：F1, F2, F3</p>
                    </div>
                </el-drawer>
            </el-row>
        </el-header>

        <el-container class="aside-main-container">
            <el-aside width="400px">
                <div class="image-container">
                    <el-image class="bqbImg"
                              :src="picUrl"
                              fit="cover"
                              :preview-src-list="picUrlList"
                              alt="pic"
                    ></el-image>
                    <div class="picTitle">{{picName}}</div>
                </div>
            </el-aside>

            <el-container>
                <el-main>
                    <el-form ref="picInfo" label-width="80px" label-position="left"
                             @keyup.ctrl.219.native.prevent="goPrevPic"
                             @keyup.ctrl.221.native.prevent="goNextPic"
                             @keyup.ctrl.enter.native.prevent="submitAndNext"
                             @keyup.112.native.prevent="picInfo.textType = '图文结合'"
                             @keyup.113.native.prevent="picInfo.textType = '无文字'"
                             @keyup.114.native.prevent="picInfo.textType = '纯文字'">
                        <el-form-item label="图片号码">
                            <el-input v-model="gotoName" class="input-short" prefix-icon="el-icon-setting"
                                      @keyup.enter.native="goPic"></el-input>
                            <el-button type="primary" @click="goPic">GO !</el-button>
                            <el-button type="primary" plain @click="goPrevPic">上一张</el-button>
                            <el-button type="primary" plain @click="goNextPic">下一张</el-button>
                        </el-form-item>
                        <el-form-item label="文字类型" v-show="config.visible.textType">
                            <el-radio-group v-model="picInfo.textType">
                                <el-radio label="图文结合"></el-radio>
                                <el-radio label="无文字"></el-radio>
                                <el-radio label="纯文字"></el-radio>
                            </el-radio-group>
                        </el-form-item>
                        <el-form-item label="角色" v-show="config.visible.role">
                            <!--<el-autocomplete id="role-input" v-model="input.roleInput"
                                             @keyup.enter.native="addRoleTag()"
                                             class="input-short" prefix-icon="el-icon-s-custom"
                                             :fetch-suggestions="querySearch"
                                             @select="addRoleTag"
                            ></el-autocomplete>-->
                            <el-input id="role-input" v-model="input.roleInput"
                                      @keyup.enter.native="addRoleTag()"
                                      class="input-short" prefix-icon="el-icon-s-custom"></el-input>
                            <el-tag
                                    v-for="roleTag in picInfo.role"
                                    :key="roleTag"
                                    closable
                                    disable-transitions=false
                                    @close="closeTag(roleTag,1)"
                            >{{roleTag}}
                            </el-tag>
                        </el-form-item>
                        <el-form-item label="情绪" v-show="config.visible.emotion">
                            <el-input id="emotion-input" v-model="input.emotionInput"
                                      @keyup.enter.native="addEmotionTag()"
                                      class="input-short" prefix-icon="el-icon-magic-stick"></el-input>
                            <el-tag
                                    v-for="emotionTag in picInfo.emotion"
                                    :key="emotionTag"
                                    closable
                                    disable-transitions=false
                                    @close="closeTag(emotionTag, 2)"
                            >{{emotionTag}}
                            </el-tag>
                        </el-form-item>
                        <el-form-item id="style-input" label="风格" v-show="config.visible.style">
                            <el-input v-model="input.styleInput" @keyup.enter.native="addStyleTag()"
                                      class="input-short" prefix-icon="el-icon-lollipop"></el-input>
                            <el-tag
                                    v-for="styleTag in picInfo.style"
                                    :key="styleTag"
                                    closable
                                    disable-transitions=false
                                    @close="closeTag(styleTag, 3)"
                            >{{styleTag}}
                            </el-tag>
                        </el-form-item>
                        <el-form-item id="topic-input" label="主题" v-show="config.visible.topic">
                            <el-input v-model="input.topicInput" @keyup.enter.native="addTopicTag()"
                                      class="input-short" prefix-icon="el-icon-s-flag"></el-input>
                            <el-tag
                                    v-for="topicTag in picInfo.topic"
                                    :key="topicTag"
                                    closable
                                    disable-transitions=false
                                    @close="closeTag(topicTag, 4)"
                            >{{topicTag}}
                            </el-tag>
                        </el-form-item>
                        <el-form-item label="描述" v-show="config.visible.description">
                            <el-input id="description-input" type="textarea" v-model="picInfo.description"></el-input>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" @click="submitAndNext">提交 & Next</el-button>
                            <el-button type="primary" @click="submit">提交</el-button>
                            <el-button @click="getInfo">从数据库同步</el-button>
                        </el-form-item>
                    </el-form>
                </el-main>
            </el-container>
        </el-container>
    </el-container>
</template>

<script>
    function makeName(num) {
        return num.toString().padStart(4, '0');
    }

    export default {
        name: 'Indexer',
        data() {
            return {
                password: '',
                config: {
                    admin: '',
                    visible: {
                        textType: true,
                        role: true,
                        emotion: true,
                        style: true,
                        topic: true,
                        description: true
                    }
                },
                picName: '',
                picInfo: {
                    textType: '',
                    role: [],
                    emotion: [],
                    style: [],
                    topic: [],
                    description: ''
                },
                input: {
                    roleInput: '', // 接下来四个是输入信息
                    emotionInput: '',
                    styleInput: '',
                    topicInput: '',
                },
                gotoName: '0001', // 下一个图片的输入编号
                helpInfoVisible: false, // 是否显示帮助抽屉
                settingVisible: false,
                loginVisible: false,
                ac: { // 自动补全信息的数据
                    roles: [
                        {value: '猫和老鼠'},
                        {value: '熊猫头'},
                        {value: '黄脸'}
                    ]
                }
            };
        },
        computed: {
            picUrlList() {
                return [this.picUrl];
            },
            picUrl() {
                return '/bqbsrc/' + this.picName;
            }
        },
        methods: {
            getInfo() {
                this.$axios({
                    method: "GET",
                    url: "./php/getPicInfo.php",
                    params: {
                        getData: {
                            picName: this.picName
                        }
                    }
                }).then(response => {
                    if (response.data['successCode'] === 0) {
                        this.picInfo = response.data['data']['picInfo'];
                        this.gotoName = this.picName.substr(0, 4);
                        console.log(this.picInfo)
                    } else {
                        console.log('getInfo response fail!')
                    }
                }).catch(error => {
                    this.$message({
                        message: '从数据库获取数据失败！',
                        type: 'error',
                        duration: 0,
                        showClose: true
                    });
                    console.log(error, "error");
                });
            },
            submit() {
                if (this.config.visible.textType && !this.picInfo.textType) {
                    this.nullAlert('文字类型');
                    return;
                } else if (this.config.visible.role && this.picInfo.role.length === 0) {
                    this.nullAlert('角色');
                    return;
                } else if (this.config.visible.emotion && this.picInfo.emotion.length === 0) {
                    this.nullAlert('情绪');
                    return;
                } else if (this.config.visible.style && this.picInfo.style.length === 0) {
                    this.nullAlert('风格');
                    return;
                } else if (this.config.visible.topic && this.picInfo.topic.length === 0) {
                    this.nullAlert('主题');
                    return;
                }
                this.$axios({
                    method: "GET",
                    url: "./php/insertPicInfo.php",
                    params: {
                        getData: {
                            picInfo: this.picInfo,
                            picName: this.picName
                        }
                    }
                }).then(response => {
                    if (response.data['successCode'] === 0) {
                        this.$message({
                            message: '更新成功！',
                            type: 'success',
                            duration: 800
                        });
                        console.log('submit success!', response);
                    } else {
                        this.$message({
                            message: '更新失败！',
                            type: 'error',
                            duration: 0,
                            showClose: true
                        });
                        console.log('submit response fail!')
                    }
                }).catch(error => {
                    alert("submit fail!");
                    console.log(error, "error");
                });

                return false;
            },
            submitAndNext() {
                this.submit();
                this.goNextPic();
            },
            nullAlert(nullItem) {
                this.$alert(nullItem + '的值不可为空！', '空值 T_T', {
                    confirmButtonText: '确定'
                });
            },
            goPrevPic() {
                let oldName = parseInt(this.picName.slice(0, 4));
                if (oldName === 1) {
                    this.$message({
                        message: '不存在上一个！',
                        type: 'warning',
                        duration: 800,
                    });
                    return;
                }
                oldName--;
                let newName = makeName(oldName);
                this.picName = newName + '.jpg';
                this.gotoName = newName;
                this.getInfo();
                this.setFocus();
            },
            goNextPic() {
                let oldName = parseInt(this.picName.slice(0, 4));
                if (oldName === 4000) {
                    this.$message({
                        message: '不存在下一个！',
                        type: 'warning',
                        duration: 800,
                    });
                    return;
                }
                oldName++;
                let newName = makeName(oldName);
                this.picName = newName + '.jpg';
                this.gotoName = newName;
                this.getInfo();
                this.setFocus();
            },
            goPic() {
                let gotoNum = parseInt(this.gotoName);
                if (gotoNum >= 1 && gotoNum <= 4000) {
                    let newName = makeName(gotoNum);
                    this.picName = newName + '.jpg';
                    this.gotoName = newName;
                    this.getInfo();
                } else {
                    this.$message({
                        message: '不存在此图片！',
                        type: 'warning',
                        duration: 800,
                    });
                }
            },
            closeTag(tag, flag) {
                let list = [];
                switch (flag) {
                    case 1:
                        list = this.picInfo.role;
                        break;
                    case 2:
                        list = this.picInfo.emotion;
                        break;
                    case 3:
                        list = this.picInfo.style;
                        break;
                    case 4:
                        list = this.picInfo.topic;
                        break;
                    default:
                }
                let index = list.indexOf(tag);
                list.splice(index, 1);
            },
            addRoleTag() {
                let input = this.input.roleInput.trim();
                if (this.picInfo.role.indexOf(input) !== -1) {
                    this.tagRepeat();
                    return;
                }
                if (input !== '') {
                    this.picInfo.role.push(input);
                }
                this.input.roleInput = '';
            },
            addEmotionTag() {
                let input = this.input.emotionInput.trim();
                if (this.picInfo.emotion.indexOf(input) !== -1) {
                    this.tagRepeat();
                    return;
                }
                if (input !== '') {
                    this.picInfo.emotion.push(input);
                }
                this.input.emotionInput = '';
            },
            addStyleTag() {
                let input = this.input.styleInput.trim();
                if (this.picInfo.style.indexOf(input) !== -1) {
                    this.tagRepeat();
                    return;
                }
                if (input !== '') {
                    this.picInfo.style.push(input);
                }
                this.input.styleInput = '';
            },
            addTopicTag() {
                let input = this.input.topicInput.trim();
                if (this.picInfo.topic.indexOf(input) !== -1) {
                    this.tagRepeat();
                    return;
                }
                if (input !== '') {
                    this.picInfo.topic.push(input);
                }
                this.input.topicInput = '';
            },
            // reset() {
            //     this.picInfo.role = [];
            //     this.picInfo.emotion = [];
            //     this.picInfo.style = [];
            //     this.picInfo.topic = [];
            //     this.picInfo.description = '';
            // },
            createFilter(queryString) {
                return (role) => {
                    return (role.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0);
                };
            },
            querySearch(queryString, cb) {
                let roles = this.ac.roles;
                let res = queryString ? roles.filter(this.createFilter((queryString))) : roles;
                cb(res);
            },
            setFocus() {
                let firstItem = this.getFirstVisible();
                if (!firstItem) {
                    return;
                }
                let id = firstItem + '-input';
                document.getElementById(id).focus();
            },
            getFirstVisible() { // 获取第一个在页面上的输入
                let el = '';
                if (this.config.visible.role) {
                    el = 'role';
                } else if (this.config.visible.emotion) {
                    el = 'emotion';
                } else if (this.config.visible.style) {
                    el = 'style';
                } else if (this.config.visible.topic) {
                    el = 'topic';
                }
                return el;
            },
            tagRepeat() {
                this.$message({
                    message: '重复的标签！',
                    type: 'warning',
                    duration: 800,
                });
            },
            init() {
                // 判断是否登录过，如果是加载配置
                this.$axios({
                    method: "GET",
                    url: "./php/init.php",
                }).then(response => {
                    if (response.data['successCode'] === 0) {
                        console.log('init success: ', response.data);
                        if (response.data['data']['hasLogged'] === true) {
                            let config = response.data['data']['config'];
                            this.config.admin = config['admin'];
                            this.config.visible = config['visible'];
                            if (response.data['data']['lastPic']) {
                                this.picName = response.data['data']['lastPic'];
                            } else {
                                this.picName = '0001.jpg';
                            }
                            this.getInfo();
                            this.setFocus();
                        } else {
                            this.loginVisible = true;
                        }
                    } else {
                        console.log('initLogin response fail!')
                    }
                }).catch(error => {
                    alert("初始化失败！");
                    console.log(error, "error");
                });
            },
            login() {
                if (this.password === 'isrbqb') {
                    // todo 以后改成哈希
                    this.loginVisible = false;
                    this.$message({
                        message: 'Welcome！',
                        type: 'success',
                        duration: 2000,
                    });
                }
                this.$axios({
                    method: "GET",
                    url: "./php/login.php",
                    params: {
                        getData: {
                            admin: this.config.admin
                        }
                    }
                }).then(response => {
                    if (response.data['successCode'] === 0) {
                        console.log('login success!', response);
                    } else {
                        console.log('login response fail!')
                    }
                }).catch(error => {
                    alert("登录信息上传失败！");
                    console.log(error, "error");
                });
                this.getInfo();
                this.setFocus();
            },
            beforeunloadFn(e) {
                console.log(e);
                this.$axios({
                    method: "GET",
                    url: "./php/getConfig.php",
                    params: {
                        getData: {
                            config: this.config,
                            lastPic: this.picName
                        }
                    }
                }).then(response => {
                    if (response.data['successCode'] === 0) {
                        console.log('upload success!', response);
                    } else {
                        console.log('upload response fail!')
                    }
                }).catch(error => {
                    alert("上传配置失败!");
                    console.log(error, "error");
                });
            }
        },
        directives: {
            focus: {
                inserted: function (el) {
                    el.focus();
                }
            }
        },
        mounted() {
            this.init();
            window.addEventListener('beforeunload', e => this.beforeunloadFn(e))
        },
        destroyed() {
            window.removeEventListener('beforeunload', e => this.beforeunloadFn(e))
        }
    }
</script>

<style>
    .root-container {
    }

    .aside-main-container {
        width: 1000px;
        margin: 0 auto;
    }

    .el-header {
        background-color: #F6F6F6;
    }

    .header-title {
        height: 60px;
        width: 1000px;
        margin: 0 auto;
    }

    .big-title {
        font-size: 20px;
        font-weight: 500;
        height: 30px;
        line-height: 30px;
        margin: 15px 0;
        padding-left: 5px;
    }

    .admin-name {
        text-align: right;
        padding-right: 30px;
        height: 30px;
        line-height: 30px;
        margin: 15px 0;
        font-size: 16px;
    }

    .info-button {
        margin: 14px 0 14px 30px;
    }

    .el-aside {
        background-color: #ffffff;
        border-radius: 10px;
        margin-right: 10px;
    }

    .el-main {
        border-radius: 10px;
    }

    .el-footer {
        background-color: deepskyblue;
    }

    .el-form-item-hasTag {
        margin-bottom: 10px;
    }

    .hasTag {
        margin-bottom: 10px;
    }

    .el-form-item .el-tag {
        margin-right: 5px;
    }

    .bqbImg {
        width: 366px;
        margin: 15px;
    }

    .image-container {
        margin-top: 20px;
        border-radius: 15px;
        border: solid 2px #0084ff;
    }

    .picTitle {
        text-align: center;
        margin-bottom: 20px;
        color: #606266;
        letter-spacing: 3px;
    }

    .input-short {
        width: 200px;
        margin-right: 10px;
    }

    .config-dialog .el-form-item {
        margin: 10px auto;
    }

    .config-dialog .el-dialog__body {
        padding: 10px 20px 10px 60px;
    }

    .login-dialog .el-dialog__body {
        padding: 20px 450px 40px 450px;
    }

    .login-dialog .el-dialog__header {
        margin-top: 100px;
    }

    .login-dialog .el-form-item {
        width: 300px;
        margin: 10px auto;
    }

    .intro {
        padding: 20px;
    }

    .sub-intro {
        color: #999999;
    }

    .intro span {
        color: #0084ff;
    }
</style>
