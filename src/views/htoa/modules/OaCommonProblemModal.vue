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
          label="标题">
          <a-input placeholder="请输入标题" v-decorator="['title', validatorRules.title]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="问题描述">
          <a-input placeholder="请输入问题描述" v-decorator="['message', validatorRules.message]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          hidden="true"
          label="文件名称">

          <a-input  placeholder="请输入文件路径" v-decorator="['filePath', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="文件">
          <template  >
            <div style="float:left">
              <a-upload
                name="file"
                :data="{'isup':1}"
                :multiple="false"
                :action="uploadAction"
                :headers="headers"
                :showUploadList="false"
                :beforeUpload="beforeUpload"
                @change="handleChange"
              >
                <div style="display: none">
                  <a-input placeholder="请输入文件名称" v-decorator="['fileName', validatorRules.fileName]" />
                </div>

                <a-button icon="upload">{{ fileName }}</a-button>
              </a-upload>
            </div>


              <template v-if="file.filePath">
                <div style="float: left;width:104px;height:104px;margin-right: 10px;margin: -5px 8px 8px 0;">
                  <div style="width: 100%;height: 100%;position: relative;padding: 8px;" @click="pdfPreview(file.fileName)">
                    <img style="width: 100%;" src="~@/assets/pdf4.jpg">
                  </div>
                  <div  style="width: 30px;height: 30px;float: right;margin-top: -110px;margin-right: -10px;position: relative">
                    <a-tooltip title="删除并重新上传">
                      <a-icon
                        v-if="file.status!=='uploading'"
                        type="close-circle"
                        style="cursor: pointer;"
                        @click="handleClickDelFile"/>
                    </a-tooltip>
                  </div>
                </div>

              </template>

          </template>


        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="内容">
          <a-input placeholder="请输入内容" v-decorator="['content', {}]" />
        </a-form-item>
		
      </a-form>
    </a-spin>
    <pdf-preview-modal ref="pdfmodal"></pdf-preview-modal >
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import Vue from 'vue'
  import { ACCESS_TOKEN } from "@/store/mutation-types"
  import PdfPreviewModal from '@/views/jeecg/modules/PdfPreviewModal'

  export default {
    name: "OaCommonProblemModal",
    components:{
      PdfPreviewModal
    },
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
          title:{ rules : [{required : true ,message : "请输入标题!"}]},
          message:{ rules : [{required : true ,message : "请输入问题描述!"}]},
          fileName:{ rules : [{required : true ,message : "请上传pdf文件!"}]}
        },
        url: {
          add: "/problem/oaCommonProblem/add",
          edit: "/problem/oaCommonProblem/edit",
          fileUpload: window._CONFIG['domianURL']+"/sys/common/upload",
        },
        // file 信息
        uploadValues: {},
        uploadLoading:false,
        fileName :"点击上传",
        file:{status:"",percent:0,id:"",fileName:"",filePath:""},
        headers:{},
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token":token }
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'title','message','fileName','filePath','content'))
		  //时间格式化
        });
        if(this.model.fileName){
          this.fileName = this.model.fileName;
          this.file.fileName = this.model.fileName;
          this.file.filePath = this.model.filePath;
        }else{
          this.fileName = "点击上传";
          this.file.filePath = "";
        }

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
      uploadAction:function () {
        return this.url.fileUpload;
      },
      beforeUpload: function(file){
        var fileType = file.type;
        if(fileType.indexOf('pdf')<0){
          this.$message.warning('请上传pdf文件');
          return false;
        }
        //TODO 验证文件大小
      },
      handleChange (info) {
        debugger
        this.file = info.file;
        console.log(this.file);
        this.percent = info.file.percent;
        if (info.file.status === 'uploading') {
          this.uploadLoading = true
          return
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;

          console.log(response);
          if(response.success){
            debugger
            this.fileName = response.result.fileName;
            this.model.fileName = response.result.fileName;
            this.model.filePath = response.result.filePath;
            this.file.fileName = response.result.fileName;
            this.file.filePath = response.result.filePath;
            this.form.setFieldsValue(pick(this.model,'fileName'))
            this.form.setFieldsValue(pick(this.model,'filePath'))
          }else{
            this.$message.warning(response.message);
          }
        }
      },
      handleClickDelFile() {
        this.fileName = "点击上传";
        this.model.fileName = "";
        this.model.filePath = "";
        this.file.fileName = "";
        this.file.filePath = "";
        this.form.setFieldsValue(pick(this.model,'fileName'))
        this.form.setFieldsValue(pick(this.model,'filePath'))
      },
      pdfPreview(title){
        const token = Vue.ls.get(ACCESS_TOKEN);
        this.headers = {"X-Access-Token":token}
        //this.PdfPreviewModal.previewFiles(title,token);
        this.$refs.pdfmodal.previewFiles(title,token);
      },

    }
  }
</script>

<style lang="less" scoped>

</style>