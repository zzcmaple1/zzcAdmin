<template>
    <!-- 对话框表单 -->
    <el-dialog
        class="ba-operate-dialog"
        top="5vh"
        :close-on-click-modal="false"
        :model-value="baTable.form.operate ? true : false"
        @close="baTable.toggleForm"
        :destroy-on-close="true"
    >
        <template #header>
            <div class="title" v-drag="['.ba-operate-dialog', '.el-dialog__header']" v-zoom="'.ba-operate-dialog'">
                {{ baTable.form.operate ? t(baTable.form.operate) : '' }}
            </div>
        </template>
        <el-scrollbar v-loading="baTable.form.loading" class="ba-table-form-scrollbar">
            <div
                class="ba-operate-form"
                :class="'ba-' + baTable.form.operate + '-form'"
                :style="'width: calc(100% - ' + baTable.form.labelWidth! / 2 + 'px)'"
            >
                <el-form
                    ref="formRef"
                    @keyup.enter="baTable.onSubmit(formRef)"
                    :model="baTable.form.items"
                    label-position="right"
                    :label-width="baTable.form.labelWidth + 'px'"
                    :rules="rules"
                    v-if="!baTable.form.loading"
                >
                    <FormItem
                        type="remoteSelect"
                        prop="pid"
                        :label="t('auth.menu.Superior menu rule')"
                        v-model="baTable.form.items!.pid"
                        :placeholder="t('Click Select')"
                        :input-attr="{
                            params: { isTree: true },
                            field: 'title',
                            'remote-url': baTable.api.actionUrl.get('index'),
                        }"
                    />
                    <el-form-item :label="t('auth.menu.Rule type')">
                        <el-radio-group v-model="baTable.form.items!.type">
                            <el-radio class="ba-el-radio" label="route" :border="true">{{ t('user.rule.Normal routing') }}</el-radio>
                            <el-radio class="ba-el-radio" label="menu_dir" :border="true">{{ t('user.rule.Member center menu contents') }}</el-radio>
                            <el-radio class="ba-el-radio" label="menu" :border="true">{{ t('user.rule.Member center menu items') }}</el-radio>
                            <el-radio class="ba-el-radio" label="nav" :border="true">{{ t('user.rule.Top bar menu items') }}</el-radio>
                            <el-radio class="ba-el-radio" label="button" :border="true">{{ t('user.rule.Page button') }}</el-radio>
                            <el-radio class="ba-el-radio" label="nav_user_menu" :border="true">{{ t('user.rule.Top bar user dropdown') }}</el-radio>
                        </el-radio-group>
                        <div class="block-help">{{ t('user.rule.Type ' + baTable.form.items!.type + ' tips') }}</div>
                    </el-form-item>
                    <el-form-item prop="title" :label="t('auth.menu.Rule title')">
                        <el-input
                            v-model="baTable.form.items!.title"
                            type="string"
                            :placeholder="t('Please input field', { field: t('auth.menu.Rule title') })"
                        ></el-input>
                    </el-form-item>
                    <el-form-item prop="name" :label="t('auth.menu.Rule name')">
                        <el-input v-model="baTable.form.items!.name" type="string" :placeholder="t('user.rule.English name')"></el-input>
                        <div class="block-help">
                            {{ t('auth.menu.It will be registered as the web side routing name and used as the server side API authentication') }}
                        </div>
                    </el-form-item>
                    <el-form-item :label="t('auth.menu.Routing path')">
                        <el-input v-model="baTable.form.items!.path" type="string" :placeholder="t('user.rule.Web side routing path')"></el-input>
                    </el-form-item>

                    <!-- 规则图标 -->
                    <FormItem
                        v-if="!['nav', 'button', 'nav_user_menu'].includes(baTable.form.items!.type)"
                        type="icon"
                        :label="t('auth.menu.Rule Icon')"
                        v-model="baTable.form.items!.icon"
                        :input-attr="{ 'show-icon-name': true }"
                    />
                    <!-- 菜单类型：tab、link、iframe -->
                    <FormItem
                        v-if="!['menu_dir', 'button', 'route'].includes(baTable.form.items!.type)"
                        :label="t('auth.menu.Menu type')"
                        v-model="baTable.form.items!.menu_type"
                        type="radio"
                        :data="{
                            content: { tab: t('auth.menu.Menu type tab'), link: t('auth.menu.Menu type link (offsite)'), iframe: 'Iframe' },
                            childrenAttr: { border: true },
                        }"
                    />
                    <!-- URL -->
                    <el-form-item
                        prop="url"
                        v-if="!['menu_dir', 'button', 'route'].includes(baTable.form.items!.type) && baTable.form.items!.menu_type != 'tab'"
                        :label="t('auth.menu.Link address')"
                    >
                        <el-input
                            v-model="baTable.form.items!.url"
                            type="string"
                            :placeholder="t('auth.menu.Please enter the URL address of the link or iframe')"
                        ></el-input>
                    </el-form-item>
                    <!-- 组件路径 -->
                    <el-form-item
                        v-if="baTable.form.items!.type == 'route' || (!['menu_dir', 'button'].includes(baTable.form.items!.type) && baTable.form.items!.menu_type == 'tab')"
                        :label="t('auth.menu.Component path')"
                    >
                        <el-input
                            v-model="baTable.form.items!.component"
                            type="string"
                            :placeholder="t('user.rule.For example, if you add account/overview as a route only')"
                        ></el-input>
                    </el-form-item>
                    <!-- 扩展属性 -->
                    <el-form-item
                        v-if="!['menu_dir', 'button'].includes(baTable.form.items!.type) && baTable.form.items!.menu_type == 'tab'"
                        :label="t('auth.menu.Extended properties')"
                    >
                        <el-select
                            class="w100"
                            v-model="baTable.form.items!.extend"
                            :placeholder="t('Please select field', { field: t('auth.menu.Extended properties') })"
                        >
                            <el-option :label="t('auth.menu.none')" value="none"></el-option>
                            <el-option :label="t('auth.menu.Add as route only')" value="add_rules_only"></el-option>
                            <el-option :label="t('auth.menu.Add as menu only')" value="add_menu_only"></el-option>
                        </el-select>
                        <div class="block-help">
                            {{ t('user.rule.Web side component path, please start with /src, such as: /src/views/frontend/index') }}
                        </div>
                    </el-form-item>
                    <FormItem
                        v-if="!['menu_dir', 'menu', 'nav_user_menu'].includes(baTable.form.items!.type)"
                        :label="t('user.rule.no_login_valid')"
                        v-model="baTable.form.items!.no_login_valid"
                        type="radio"
                        :data="{
                            content: { '0': t('user.rule.no_login_valid 0'), '1': t('user.rule.no_login_valid 1') },
                            childrenAttr: { border: true },
                        }"
                        :attr="{
                            'block-help': t('user.rule.no_login_valid tips'),
                        }"
                    />
                    <el-form-item :label="t('auth.menu.Rule comments')">
                        <el-input
                            @keyup.enter.stop=""
                            @keyup.ctrl.enter="baTable.onSubmit(formRef)"
                            v-model="baTable.form.items!.remark"
                            type="textarea"
                            :autosize="{ minRows: 2, maxRows: 5 }"
                            :placeholder="t('Please input field', { field: t('auth.menu.Rule comments') })"
                        ></el-input>
                    </el-form-item>
                    <el-form-item :label="t('auth.menu.Rule weight')">
                        <el-input
                            v-model="baTable.form.items!.weigh"
                            type="number"
                            :placeholder="t('auth.menu.Please enter the weight of menu rule (sort by)')"
                        ></el-input>
                    </el-form-item>
                    <FormItem
                        :label="t('state')"
                        v-model="baTable.form.items!.status"
                        type="radio"
                        :data="{
                            content: { '0': t('Disable'), '1': t('Enable') },
                            childrenAttr: { border: true },
                        }"
                    />
                </el-form>
            </div>
        </el-scrollbar>
        <template #footer>
            <div :style="'width: calc(100% - ' + baTable.form.labelWidth! / 1.8 + 'px)'">
                <el-button @click="baTable.toggleForm('')">{{ t('Cancel') }}</el-button>
                <el-button v-blur :loading="baTable.form.submitLoading" @click="baTable.onSubmit(formRef)" type="primary">
                    {{ baTable.form.operateIds && baTable.form.operateIds.length > 1 ? t('Save and edit next item') : t('Save') }}
                </el-button>
            </div>
        </template>
    </el-dialog>
</template>

<script setup lang="ts">
import { reactive, ref, inject } from 'vue'
import { useI18n } from 'vue-i18n'
import type baTableClass from '/@/utils/baTable'
import FormItem from '/@/components/formItem/index.vue'
import type { ElForm, FormItemRule } from 'element-plus'
import { buildValidatorData } from '/@/utils/validate'

const formRef = ref<InstanceType<typeof ElForm>>()
const baTable = inject('baTable') as baTableClass

const { t } = useI18n()

const rules: Partial<Record<string, FormItemRule[]>> = reactive({
    title: [buildValidatorData({ name: 'required', title: t('auth.menu.Rule title') })],
    pid: [
        {
            validator: (rule: any, val: string, callback: Function) => {
                if (!val) {
                    return callback()
                }
                if (parseInt(val) == parseInt(baTable.form.items!.id)) {
                    return callback(new Error(t('auth.menu.The superior menu rule cannot be the rule itself')))
                }
                return callback()
            },
            trigger: 'blur',
        },
    ],
    name: [buildValidatorData({ name: 'required', title: t('auth.menu.Rule name') })],
    url: [buildValidatorData({ name: 'url', message: t('auth.menu.Please enter the correct URL') })],
})
</script>

<style scoped lang="scss">
.ba-el-radio {
    margin-bottom: 10px;
}
</style>
