<template>
<!--  <layout name="LayoutDefault">-->
    <div class="advices">
      <a-row :gutter="10">
        <a-col
          :xs="24"
          :sm="24"
          :md="18"
          :lg="18"
          :xl="18">
          <template v-if="advices.length">
            <div class="left">
              <a-skeleton :loading="loading">
                <a-list
                  :pagination="pagination"
                  :data-source="advices"
                  item-layout="vertical"
                  size="large"
                  class="advice-list"
                >
                  <a-list-item
                    slot="renderItem"
                    slot-scope="item, index"
                    key="item.title"
                  >
                    <template
                      v-for="{ type, text } in actions"
                      slot="actions">
                      <span :key="type">
                        <a-icon
                          :type="type"
                          style="margin-right: 8px"
                          @click="like()"/>
                        {{ text }}
                      </span>
                    </template>
                    <a-list-item-meta :description="item.description">
                      <a
                        slot="title"
                        :href="item.href">{{ item.title }}</a>
                      <a-avatar
                        slot="avatar"
                        :src="item.avatar" />
                    </a-list-item-meta>
                    {{ item.content }}
                  </a-list-item>
                </a-list>
              </a-skeleton>
            </div>
          </template>
          <template v-else>
          <div class="left advices-none">
            <p>您还没有提交建议。</p>
            <p>
              您可以
              <router-link to="/advice/add">写一条建议</router-link>，也可以查看
              <router-link to="/advice">常见建议</router-link>。
            </p>
          </div>
          </template>
        </a-col>
        <a-col
          :xs="24"
          :sm="24"
          :md="24"
          :lg="6"
          :xl="6">
          <div class="advice-example right">
            <a-button
              :size="large"
              type="primary"
              icon="edit"
              block
              @click="add()"
            >我要提建议</a-button
            >
            <div class="advice-notice">
              <p class="letter-bd">请详细描述您的问题以及建议</p>
              <p>我们会细心聆听，解决问题！</p>
            </div>
            <a-skeleton :loading="loading">
              <a-list
                :data-source="examples"
                bordered
                class="advice-example-list">
                <a-list-item
                  slot="renderItem"
                  slot-scope="item, index">
                  <a :href="item.filePath" target="_blank">{{ item.title }}</a>
                </a-list-item>
                <div
                  slot="header"
                  class="advice-example-title">
                  以下咨询可能对你有帮助！
                </div>
              </a-list>
            </a-skeleton>
          </div>
        </a-col>
      </a-row>

    </div>
<!--  </layout>-->
</template>

<script>
  // import Layout from '../layouts/Layout';

  export default {
    name: `my`,
    components: {
      // Layout,
    },
    data() {
      return {
        loading:true,
        large: `large`,
        examples: [],
        advices: [],
        pagination: {
          onChange: (page) => {
            console.log(page);
          },
          pageSize: 10,
        },
        actions: [
          { type: `like`, text: `156` },
          { type: `dislike`, text: `156` },
        ],
      };
    },
    created(){
      this.getAdvices();
      this.getExamples()
    },
    methods:{
      add(){
        this.$router.push('/advice/add')
      },
      // 获取帮助列表
      getExamples() {
        const _this = this;
        this.axios(`api/queryProblemlist`).then((res) => {
          if (res.code === 0) {
            this.loading = false;
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
      //  获取建议列表
      getAdvices() {
        const _this = this;
        this.axios(`api/queryFeedbackPageList`).then((res) => {
          if (res.code === 0) {
            this.loading = false;
            res.result.records.map((item,index)=>{
              if(item.avatar){
                item.avatar = window._CONFIG['domianURL'] + "/" +item.avatar
              }
              return item
            })
            _this.$store.commit(`SAVE_ADVICES`, res.result.records);
            this.advices = JSON.parse(localStorage.getItem(`advices`)) || this.$store.state.advices;
          }
        });
      },
    }
  };
</script>
<style>
  .advices-none{
    text-align: center;
  }
  .advice-example-list{
    background-color: #f1f1f1 !important;
  }
  <style>
