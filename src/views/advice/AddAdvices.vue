<template>
<!--  <layout name="LayoutDefault">-->
  <a-row :gutter="10">
    <a-col :xs="24" :sm="24" :md="18" :lg="18" :xl="18">
      <div class="left">
        <p class="advice-title">
          您的建议我们将全力处理，如有咨询或疑问需要解决，请进入
          <router-link to="/advice">常见建议</router-link>
        </p>
       <a-form :form="form">
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="问题类别">
          <j-dict-select-tag v-decorator="['modul', validatorRules.modul]" :triggerChange="true" placeholder="请选择问题类别" dictCode="oa_modul_dict,modul_name,modul_code"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="问题描述">
          <a-textarea rows="8" placeholder="请输入问题描述" v-decorator="['content', validatorRules.content]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          :hidden="true"
          label="图片">
          <a-input placeholder="请输入图片" v-decorator="['image', {}]"  />
        </a-form-item>


        <a-form-item label="问题截图" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <div>
            <a-row>
              <!-- 预览区域 -->
              <a-col :span="12" style="width:100%" >
                <template>
                  <div v-for="(file,index) in files" :key="index">
                    <div style="float: left;width:104px;height:104px;margin-right: 10px;margin: 0 8px 8px 0;">
                      <div style="width: 100%;height: 100%;position: relative;padding: 8px;border: 1px solid #d9d9d9;border-radius: 4px;">
                        <img style="width: 100%;" :src="url.imgerver + file.filePath"  preview="0">
                      </div>
                      <div  style="width: 30px;height: 30px;float: right;margin-top: -116px;margin-right: -14px;position: relative">
                        <a-tooltip title="删除">
                          <a-icon
                            type="close-circle"
                            style="cursor: pointer;"
                            @click="handleClickDelFile(file.id,index)"/>
                        </a-tooltip>
                      </div>
                    </div>
                  </div>
                </template>
                <a-upload
                  listType="picture-card"
                  class="avatar-uploader"
                  :showUploadList="false"
                  :action="uploadAction"
                  :data="uploadData"
                  :headers="headers"
                  :beforeUpload="beforeUpload"
                  @change="handleChange"
                >
                  <img v-if="picUrl" :src="getAvatarView()" alt="图片" style="height:104px;max-width:300px" />

                  <div v-else>
                    <a-icon :type="uploadLoading ? 'loading' : 'plus'" />
                    <div class="ant-upload-text">上传</div>
                  </div>
                </a-upload>
              </a-col>
            </a-row>
          </div>


        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="联系电话">
          <a-input placeholder="请输入联系电话" v-decorator="['telephone',  validatorRules.telephone]" />
        </a-form-item>
        
        <a-form-item
        :wrapperCol="{ span: 24 }"
        style="text-align: center"
      >
        <a-button htmlType="submit" type="primary" @click="handleSubmit">提交</a-button>
        <a-button style="margin-left: 8px">保存</a-button>
      </a-form-item>

      </a-form>
      </div>
    </a-col>
    <a-col :xs="24" :sm="24" :md="24" :lg="6" :xl="6">
      <div class="advice-example right">
        <a-list :data-source="examples" bordered>
          <a-list-item slot="renderItem" slot-scope="item, index">
            <a :href="item.filePath" target="_blank">{{ item.title }}</a>
          </a-list-item>
          <div slot="header" class="advice-example-title">以下咨询可能对你有帮助！</div>
        </a-list>
      </div>
    </a-col>
  </a-row>
  </div>
  <!--  </layout>-->
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import { disabledAuthFilter } from "@/utils/authFilter"
  import { ACCESS_TOKEN } from "@/store/mutation-types"
  import Vue from 'vue'
  import { getAction} from '@/api/manage'
  import { deleteAction} from '@/api/manage'
  import { postAction } from '@/api/manage'

  export default {
    name: `add`,
    components: {
      // Layout,
    },
    data() {
      return {
        formItemLayout: {
          type: '',
          content: ''
        },
        headers: {},
        uploadData: {
          'isup': 1,
          entityName: "oaFeedBack",
          entityId: ""
        },
        advicetypes: [],
        examples: [],
        previewVisible: false,
        previewImage: '',
        fileList: [{
        }],
        title:"操作",
        visible: false,
        imageData:"123",
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        uploadLoading:false,
        headers:{},
        picUrl: "",
        uploadData: {'isup':1 ,entityName : "oaFeedBack" ,entityId : ""},
        files:[],
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
          modul:{ rules : [{required : true ,message : "请选择问题类别!"}]},
          etype:{ rules : [{required : true ,message : "请输入问题类型!"}]},
          content:{ rules : [{required : true ,message : "请输入问题描述!"}]},
          telephone:{ rules : [
                  {required : true ,message : "请输入联系电话!"},
                  {validator:function(rule,value,callback){
                    if(/^1[34578]\d{9}$/.test(value) == false){
                      callback(new Error("请输入正确的手机号"));
                    }else{
                      callback();
                    }
                  }, trigger: 'blur'}
              ]}
        },
        url: {
          add: "/api/addFeedback",
          fileUpload: window._CONFIG['domianURL']+"/sys/common/upload",
          imgerver: window._CONFIG['domianURL']+"/sys/common/view/",
          pdferver: window._CONFIG['domianURL']+"/sys/common/viewPDF/",
          
        }
      };
    },
   created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token":token }
      this.getExamples();
    },
    computed:{
      uploadAction:function () {
        return this.url.fileUpload;
      }
    },
    methods: {
      handleSubmit () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            httpurl+=this.url.add;
            method = 'post';
            
            let formData = Object.assign(this.model, values);
            //时间格式化
            console.log(formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
              }else{
                that.$message.warning(res.message);
                
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
        })
      },
      normFile  (e) {
        console.log('Upload event:', e)
        if (Array.isArray(e)) {
          return e
        }
        return e && e.fileList
      },
      beforeUpload: function(file){
        var fileType = file.type;
        if(fileType.indexOf('image')<0){
          this.$message.warning('请上传图片');
          return false;
        }
        //TODO 验证文件大小
      },
      handleChange (info) {
        this.picUrl = "";
        if (info.file.status === 'uploading') {
          this.uploadLoading = true
          return
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;

          console.log(response);
          if(response.success){
            this.model.avatar = response.message;
            this.uploadData.entityId = response.result.entityId;
            this.model.image = response.result.entityId;
            this.form.setFieldsValue(pick(this.model,'image'))
            this.files.push(response.result);
            //this.picUrl = "Has no pic url yet";

          }else{
            this.$message.warning(response.message);
          }
        }
      },
      handleClickDelFile(id,index) {
        if(this.model.image){
          let that = this
          let params = {
            id: id
          };
          deleteAction("/attachment/sysAttachment/delete",params).then((res)=>{
            if(res.success){
              //this.files = res.result;
              this.files.splice(index,1)
            }
          })
        }
      },
      getAvatarView(){
        return this.url.imgerver + this.model.avatar;
      },
      getFileList () {
        this.files =[];
        if(this.model.image){
          let that = this
          let params = {
            entityId: this.model.image,
            entityName: 'oaFeedback'
          };
          getAction("/attachment/sysAttachment/queryByEntity",params).then((res)=>{
            if(res.success){
              this.files = res.result;
            }
          })
        }

      },
      // 获取帮助列表
      getExamples() {
        const _this = this;
        this.axios(`api/queryProblemlist`).then((res) => {
          if (res.code === 0) {
            res.result.records.map((item,index)=>{
              if(item.filePath){
                item.filePath = window._CONFIG['pdfDomain'] + "/" +item.filePath
              }
              return item
            })
            _this.$store.commit(`SAVE_EXAMPLES`, res.result.records);
            this.examples = JSON.parse(localStorage.getItem(`examples`)) ||
              this.$store.state.examples;
          } else {
            this.$message.error(`数据获取失败`);
          }
        });
      },
    }
  };</script>
<style>
  .left{
    padding:20px;
    background-color: #fff;
    border-radius: 5px;
  }
  .advice-title {
    font-size: 1.2em;
    color: #000;
    font-weight: 700;
    text-align: center;
    padding: 40px 0;
    border-bottom: 1px dashed #dbdcdc;
  }
  .submit{
    border-top: 1px dashed #dbdcdc;
  }
  .right{
    background-color: #f1f1f1;
    border-radius: 5px;
  }
  .advice-example-title{
    font-size: 16px;
    color: #000;
  }
  .ant-list-split .ant-list-item {
    border-bottom: none
  }
</style>
