<template lang="pug">
.bg-white.px-3.pt-3.pb-1.formData-editor
  h3.mb-4 基础信息
  form.form-horizontal(
    v-if="isLoaded",
    :action="API + 'cover/'",
    method="post",
    @submit.prevent="save"
  )
    .form-group.row
      label.col-md-2.form-control-label(for="label") 封面图名称
      .col-md-6
        input.form-control(
          id="label",
          name="label",
          placeholder="名称",
          v-model="formData.label",
          required,
          maxlength="100",
        )
    .form-group.row
      label.col-md-2.form-control-label(for="path") 长度
      .col-md-6
        uploader#path.form-group(
          v-model="formData.path",
          file-type="图片",
          file-icon="fa-file-image-o",
          mime-type="image/*",
          label="请上传 jpg png 格式的图片文件",
          errorMessage="只能上传 JPG、PNG 格式的图片文件。",
          type="cover",
          :extension="extensionReg",
        )
    .form-group.row
      label.col-md-2.form-control-label(for="start_at") 上线日期
      .col-md-6
        datepicker#start_at(
          v-model="formData.start_at"
          format="yyyy-MM-dd",
          name="start_at",
          placeholder="上线日期",
        )
    .form-group.row
      label.col-md-2.form-control-label(for="end_at") 下线日期
      .col-md-6
        datepicker#end_at(
          v-model="formData.end_at"
          format="yyyy-MM-dd",
          name="end_at",
          placeholder="下线日期",
        )

    hr
    .alert.offset-2(:class="'alert-' + status", v-if="message") {{message}}
    .form-group.row
      .col-sm-10.offset-2
        button.btn.btn-primary(:disabled="preventSubmit")
          i.fa.fa-spin.fa-spinner(v-if="isSaving")
          i.fa.fa-check(v-else)
          | 保存
        router-link.btn.btn-link(:to="{name: 'cover.list'}") 取消

  div(v-else)
    i.fa.fa-spin.fa-spinner.fa-3x
</template>

<script>
import {assign} from 'lodash';
import axios from 'axios';

import Datepicker from 'src/components/Datepicker.vue';
import Uploader from 'src/components/form/Uploader';
import baseMixin from 'src/mixin/base';

/* global API */

export default {
  components: {
    Datepicker,
    Uploader,
  },
  mixins: [baseMixin],

  data() {
    return {
      formData: {
        label: '',
        start_at: '',
        end_at: '',
        path: '',
      },

      API: API,
      isLoaded: true,
      isSaving: false,
      message: '',
      status: '',
      extensionReg: /\.(jpg|png)$/i,
    };
  },
  computed: {
    id() {
      return this.$route.params.id;
    },
    preventSubmit() {
      return this.isSaving || this.status === 'success';
    },
  },
  methods: {
    fetch() {
      return axios.get(`cover/${this.id}`)
        .then( json => {
          this.isLoaded = true;
          json.data.start_at = json.data.start_at.substr(0, 10);
          json.data.end_at = json.data.end_at.substr(0, 10);
          this.formData = json.data;
        })
        .catch( err => {
          alert('加载封面图失败。' + err);
        });
    },
    removeFlash(event) {
      event.target.classList.remove('flash');
    },
    save() {
      this.isSaving = true;
      let api = 'cover';
      let method = this.id ? 'patch' : 'post';
      if (this.id) {
        api += '/' + this.id;
      }
      const data = assign({}, this.formData, this.extraData);
      axios[method](api, data)
        .then( json => {
          if (json.code !== 0) {
            throw new Error('保存失败。' + json.msg);
          }

          this.message = '保存成功';
          this.status = 'success';
          setTimeout( () => {
            this.$router.push({
              name: '封面图管理'
            })
          }, 2500);
        })
        .catch( err => {
          if (err.status === 422) { // 提交数据有误，表单验证不通过
            err = '保存失败。' + err.body.map((value, key) => {
              if (value instanceof Array) {
                value = value.join('');
              }
              return value.replace(key, DICT[key]);
            }).join('; ');
          } else if (err.status === 423) {
            err = err.body.msg;
          }
          this.message = err;
          this.status = 'danger';
        })
        .then(() => {
          this.isSaving = false;
        });
    }
  },
  beforeMount() {
    if (this.id) {
      this.isLoaded = false;
      return this.fetch();
    }
  },
  watch: {
    '$route'(to) {
      this.id = to.params.id;
      this.fetch();
    },
  },
}
</script>