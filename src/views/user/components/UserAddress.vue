<template>
  <a-form
    ref="formRef"
    name="dynamic_form_item"
    :model="dynamicValidateForm"
    v-bind="formItemLayoutWithOutLabel"
  >
    <a-form-item
      v-for="(domain, index) in dynamicValidateForm.domains"
      :key="domain.id"
      v-bind="index === 0 ? formItemLayout : {}"
      :label="index === 0 ? '地址' : ''"
    >
      <!--      :name="['address', 'name', 'phone']"-->
      <!--      :rules="{-->
      <!--        required: true,-->
      <!--        message: '地址不能为空',-->
      <!--        trigger: 'change',-->
      <!--      }"-->

      <a-input
        v-model:value="domain.name"
        placeholder="请输入姓名"
        style="width: 80%; margin-top: 8px"
      />
      <a-input
        v-model:value="domain.phone"
        placeholder="请输入手机号"
        style="width: 80%; margin-top: 8px"
      />
      <a-input
        v-model:value="domain.address"
        placeholder="请输入地址"
        style="width: 80%; margin-top: 8px; margin-right: 8px"
      />
      <MinusCircleOutlined
        v-if="dynamicValidateForm.domains.length > 1"
        class="dynamic-delete-button"
        :disabled="dynamicValidateForm.domains.length === 1"
        @click="removeDomain(domain)"
      />
    </a-form-item>
    <a-form-item v-bind="formItemLayoutWithOutLabel">
      <a-button type="dashed" style="width: 80%" @click="addDomain">
        <PlusOutlined />
        添加地址
      </a-button>
    </a-form-item>
    <a-form-item v-bind="formItemLayoutWithOutLabel">
      <a-button type="primary" html-type="submit" @click="submitForm">
        保存
      </a-button>
      <a-button style="margin-left: 10px" @click="resetForm">重置</a-button>
    </a-form-item>
  </a-form>
</template>
<script>
  import { MinusCircleOutlined, PlusOutlined } from '@ant-design/icons-vue'
  import { defineComponent, onMounted, reactive, ref } from 'vue'
  import {
    createAddress,
    deleteAddress,
    getAddresses,
    updateAddress,
  } from '@/api/manage_user'
  import { message } from 'ant-design-vue'
  export default defineComponent({
    props: {
      id: Number,
    },
    components: {
      MinusCircleOutlined,
      PlusOutlined,
    },
    setup(props) {
      const formRef = ref()
      const formItemLayout = {
        labelCol: {
          xs: {
            span: 24,
          },
          sm: {
            span: 4,
          },
        },
        wrapperCol: {
          xs: {
            span: 24,
          },
          sm: {
            span: 20,
          },
        },
      }
      const formItemLayoutWithOutLabel = {
        wrapperCol: {
          xs: {
            span: 24,
            offset: 0,
          },
          sm: {
            span: 20,
            offset: 4,
          },
        },
      }
      const dynamicValidateForm = reactive({
        domains: [],
      })
      const submitForm = () => {
        formRef.value
          .validate()
          .then(() => {
            // console.log('values', dynamicValidateForm.domains)
            dynamicValidateForm.domains.forEach((item) => {
              if (item.id) {
                // update
                updateAddress(props.id, item.id, item).then(() => {
                  // console.log(resp.data)
                })
              } else {
                // create
                createAddress(props.id, item).then(() => {
                  // console.log(resp.data)
                })
              }
            })
            message.success('地址保存成功')
          })
          .catch((error) => {
            console.log('error', error)
          })
      }
      const resetForm = () => {
        formRef.value.resetFields()
      }
      const removeDomain = (item) => {
        if (item.id) {
          deleteAddress(props.id, item.id).then(() => {
            // console.log(resp.data)
            message.warn('地址删除成功')
            let index = dynamicValidateForm.domains.indexOf(item)
            if (index !== -1) {
              dynamicValidateForm.domains.splice(index, 1)
            }
          })
        } else {
          let index = dynamicValidateForm.domains.indexOf(item)
          if (index !== -1) {
            dynamicValidateForm.domains.splice(index, 1)
          }
        }
      }
      const addDomain = () => {
        dynamicValidateForm.domains.push({
          value: '',
          key: Date.now(),
        })
      }
      onMounted(() => {
        getAddresses(props.id).then((resp) => {
          dynamicValidateForm.domains = resp.data
        })
      })
      return {
        formRef,
        formItemLayout,
        formItemLayoutWithOutLabel,
        dynamicValidateForm,
        submitForm,
        resetForm,
        removeDomain,
        addDomain,
      }
    },
  })
</script>
<style>
  .dynamic-delete-button {
    cursor: pointer;
    position: relative;
    top: 4px;
    font-size: 24px;
    color: #999;
    transition: all 0.3s;
  }
  .dynamic-delete-button:hover {
    color: #777;
  }
  .dynamic-delete-button[disabled] {
    cursor: not-allowed;
    opacity: 0.5;
  }
</style>
