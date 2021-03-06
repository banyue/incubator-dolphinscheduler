/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
<template>
  <m-popup
          ref="popup"
          :ok-text="item ? $t('Edit') : $t('Submit')"
          :nameText="item ? $t('Edit User') : $t('Create User')"
          @ok="_ok">
    <template slot="content">
      <div class="create-user-model">
        <m-list-box-f>
          <template slot="name"><b>*</b>{{$t('User Name')}}</template>
          <template slot="content">
            <x-input
                    type="input"
                    v-model="userName"
                    maxlength="60"
                    :placeholder="$t('Please enter user name')">
            </x-input>
          </template>
        </m-list-box-f>
        <m-list-box-f v-if="router.history.current.name !== 'account'">
          <template slot="name"><b>*</b>{{$t('Password')}}</template>
          <template slot="content">
            <x-input
                    type="input"
                    v-model="userPassword"
                    :placeholder="$t('Please enter your password')">
            </x-input>
          </template>
        </m-list-box-f>
        <m-list-box-f v-if="isADMIN">
          <template slot="name"><b>*</b>{{$t('Tenant')}}</template>
          <template slot="content">
            <x-select v-model="tenantId">
              <x-option
                      v-for="city in tenantList"
                      :key="city.id"
                      :value="city.id"
                      :label="city.code">
              </x-option>
            </x-select>
          </template>
        </m-list-box-f>
        <m-list-box-f v-if="isADMIN">
          <template slot="name">{{$t('Queue')}}</template>
          <template slot="content">
            <x-select v-model="queueName">
              <x-input slot="trigger" slot-scope="{ selectedModel }" readonly :placeholder="$t('Please select a queue')" :value="selectedModel ? selectedModel.label : ''" style="width: 200px;" @on-click-icon.stop="queueName = ''">
                <i slot="suffix" class="ans-icon-fail-solid" style="font-size: 15px;cursor: pointer;" v-show="queueName ==''"></i>
                <i slot="suffix" class="ans-icon-arrow-down" style="font-size: 12px;" v-show="queueName!=''"></i>
              </x-input>
              <x-option
                      v-for="city in queueList"
                      :key="city.id"
                      :value="city.id"
                      :label="city.code">
              </x-option>
            </x-select>
          </template>
        </m-list-box-f>
        <m-list-box-f>
          <template slot="name"><b>*</b>{{$t('Email')}}</template>
          <template slot="content">
            <x-input
                    type="input"
                    v-model="email"
                    :placeholder="$t('Please enter email')">
            </x-input>
          </template>
        </m-list-box-f>
        <m-list-box-f>
          <template slot="name">{{$t('Phone')}}</template>
          <template slot="content">
            <x-input
                    type="input"
                    v-model="phone"
                    :placeholder="$t('Please enter phone number')">
            </x-input>
          </template>
        </m-list-box-f>
      </div>
    </template>
  </m-popup>
</template>
<script>
  import _ from 'lodash'
  import i18n from '@/module/i18n'
  import store from '@/conf/home/store'
  import router from '@/conf/home/router'
  import mPopup from '@/module/components/popup/popup'
  import mListBoxF from '@/module/components/listBoxF/listBoxF'

  export default {
    name: 'create-user',
    data () {
      return {
        store,
        router,
        queueList: [],
        userName: '',
        userPassword: '',
        tenantId: '',
        queueName: '',
        email: '',
        phone: '',
        tenantList: [],
        // Source admin user information
        isADMIN: store.state.user.userInfo.userType === 'ADMIN_USER' && router.history.current.name !== 'account'
      }
    },
    props: {
      item: Object
    },
    methods: {
      _ok () {
        if (this._verification()) {
          // The name is not verified
          if (this.item && this.item.groupName === this.groupName) {
            this._submit()
            return
          }
          // Verify username
          this.store.dispatch(`security/verifyName`, {
            type: 'user',
            userName: this.userName
          }).then(res => {
            this._submit()
          }).catch(e => {
            this.$message.error(e.msg || '')
          })
        }
      },
      _verification () {
        let regEmail = /^([a-zA-Z0-9]+[_|\-|\.]?)*[a-zA-Z0-9]+@([a-zA-Z0-9]+[_|\-|\.]?)*[a-zA-Z0-9]+\.[a-zA-Z]{2,}$/ // eslint-disable-line

        // Mobile phone number regular
        let regPhone = /^1(3|4|5|6|7|8)\d{9}$/; // eslint-disable-line
        
        let regPassword = /^(?![0-9]+$)(?![a-z]+$)(?![A-Z]+$)(?![`~!@#$%^&*()_\-+=<>?:"{}|,.\/;'\\[\]·~！@#￥%……&*（）——\-+={}|《》？：“”【】、；‘’，。、]+$)[`~!@#$%^&*()_\-+=<>?:"{}|,.\/;'\\[\]·~！@#￥%……&*（）——\-+={}|《》？：“”【】、；‘’，。、0-9A-Za-z]{6,22}$/;

        // user name
        if (!this.userName.replace(/\s*/g,"")) {
          this.$message.warning(`${i18n.$t('Please enter user name')}`)
          return false
        }
        // password
        if (this.userPassword!='' && this.item) {
          if(!regPassword.test(this.userPassword)) {
            this.$message.warning(`${i18n.$t('Password consists of at least two combinations of numbers, letters, and characters, and the length is between 6-22')}`)
            return false
          }
        } else if(!this.item){
          if(!regPassword.test(this.userPassword)) {
            this.$message.warning(`${i18n.$t('Password consists of at least two combinations of numbers, letters, and characters, and the length is between 6-22')}`)
            return false
          }
        }

        // email
        if (!this.email) {
          this.$message.warning(`${i18n.$t('Please enter email')}`)
          return false
        }
        // Verify email
        if (!regEmail.test(this.email)) {
          this.$message.warning(`${i18n.$t('Please enter the correct email format')}`)
          return false
        }
        // Verify phone
        if (this.phone) {
          if (!regPhone.test(this.phone)) {
            this.$message.warning(`${i18n.$t('Please enter the correct mobile phone format')}`)
            return false
          }
        }

        return true
      },
      _getQueueList () {
        return new Promise((resolve, reject) => {
          this.store.dispatch('security/getQueueList').then(res => {
          
            this.queueList = _.map(res, v => {
              return {
                id: v.id,
                code: v.queueName
              }
            })
            this.$nextTick(() => {
              this.queueName = this.queueList[0].id
            })
            resolve()
          })
        })
      },
      _getTenantList () {
        return new Promise((resolve, reject) => {
          this.store.dispatch('security/getTenantList').then(res => {
            let arr = _.filter(res, (o) => {
              return o.id !== -1
            })
            this.tenantList = _.map(arr, v => {
              return {
                id: v.id,
                code: v.tenantName
              }
            })
            this.$nextTick(() => {
              this.tenantId = this.tenantList[0].id
            })
            resolve()
          })
        })
      },
      _submit () {
        this.$refs['popup'].spinnerLoading = true
        let param = {
          userName: this.userName,
          userPassword: this.userPassword,
          tenantId: this.tenantId,
          email: this.email,
          queue: this.queueList.length>0? _.find(this.queueList, ['id', this.queueName]).code : '',
          phone: this.phone
        }

        if (this.item) {
          param.id = this.item.id
        }

        this.store.dispatch(`security/${this.item ? 'updateUser' : 'createUser'}`, param).then(res => {
          setTimeout(() => {
            this.$refs['popup'].spinnerLoading = false
          }, 800)
          this.$emit('onUpdate', param)
          this.$message.success(res.msg)
        }).catch(e => {
          this.$message.error(e.msg || '')
          this.$refs['popup'].spinnerLoading = false
        })
      }
    },
    watch: {},
    created () {
      // Administrator gets tenant list
      if (this.isADMIN) {
        Promise.all([this._getQueueList(), this._getTenantList()]).then(() => {
          if (this.item) {
            this.userName = this.item.userName
            this.userPassword = ''
            this.email = this.item.email
            this.phone = this.item.phone
            this.tenantId = this.item.tenantId
            this.$nextTick(() => {
              this.queueName = _.find(this.queueList, ['code', this.item.queue]).id||''
            })
          }
        })
      } else {
        if (this.item) {
          this.userName = this.item.userName
          this.userPassword = ''
          this.email = this.item.email
          this.phone = this.item.phone
          this.tenantId = this.item.tenantId
          if(this.queueList.length>0) {
            this.queueName = _.find(this.queueList, ['code', this.item.queue]).id
          } else {
            this.queueName = ''
          }
        }
      }
    },
    mounted () {

    },
    components: { mPopup, mListBoxF }
  }
</script>
