<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="模块名称">
          <j-dict-select-tag v-decorator="['modul', validatorRules.modul]" :triggerChange="true" placeholder="请选择模块名称" dictCode="oa_modul_dict,modul_name,modul_code"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          :hidden="true"
          label="问题类型">
          <j-dict-select-tag v-decorator="['etype', validatorRules.etype]" :triggerChange="true" placeholder="请选择问题类型" dictCode="exceptionType"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="问题描述">
          <a-textarea placeholder="请输入问题描述" v-decorator="['content', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          :hidden="true"
          label="图片">
          <a-input placeholder="请输入图片" v-decorator="['image', {}]"  />
        </a-form-item>


        <a-form-item label="图片" :labelCol="labelCol" :wrapperCol="wrapperCol">
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
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          hidden="true"
          label="状态">
          <j-dict-select-tag v-decorator="['state', {}]" :triggerChange="true" placeholder="请选择状态" dictCode="feedback_state"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="管理员回复">
          <a-textarea placeholder="请输入回复内容" v-decorator="['replyMsg', {}]" />
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
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
    name: "OaFeedbackModal",
    data () {
      return {
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
          modul:{ rules : [{required : true ,message : "请输入模块名称!"}]},
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
          add: "/feedback/oaFeedback/add",
          edit: "/feedback/oaFeedback/edit",
          fileUpload: window._CONFIG['domianURL']+"/sys/common/upload",
          imgerver: window._CONFIG['domianURL']+"/sys/common/view/",
        }
        ,
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token":token }
    },
    computed:{
      uploadAction:function () {
        return this.url.fileUpload;
      }
    },
    methods: {
      add () {
        this.edit({});
        this.model.state = 1;
        this.form.setFieldsValue(pick(this.model,'state'))

      },
      edit (record) {

        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.uploadData.entityId = this.model.image;
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'modul','etype','content','image','telephone','urgency','state','replyMsg'))
        });
        this.getFileList ();
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            
            console.log(formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
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
      handleCancel () {
        this.close()
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

      }
    }
  }
</script>

<style lang="less" scoped>>
.table-operator {
  margin-bottom: 10px
}

.clName .ant-tree li span.ant-tree-switcher, .ant-tree li span.ant-tree-iconEle {
  width: 10px !important;
}

.clName .ant-tree li .ant-tree-node-content-wrapper.ant-tree-node-selected {
  background-color: #1890FF !important;
}
</style>