<template>
    <a-row :class="['p-4', `${prefixCls}--box`]" type="flex" :gutter="10" style="max-height: 800px">
        <a-col :xl="6" :lg="24" :md="24" style="margin-bottom: 10px;">
            <DepartLeftTree ref="leftTree" @select="onTreeSelect"/>
        </a-col>
        <a-col :xl="18" :lg="24" :md="24" style="margin-bottom: 10px;">
            <div style="height: 100%; background-color: white;">
                <!--引用表格-->
                <BasicTable @register="registerTable">
                    <template #post="{text}">
                         {{(text || '').split(',').map(t => positionInfo[t] ? positionInfo[t] : t).join(',')}}
                    </template>
                </BasicTable>
            </div>
        </a-col>
    </a-row>
</template>

<script lang="ts" setup>
    import {provide, ref, unref} from 'vue'
    import {useDesign} from '/@/hooks/web/useDesign'
    import DepartLeftTree from './components/DepartLeftTree.vue'
    import {BasicTable} from '/@/components/Table';
    import {useListPage} from '/@/hooks/system/useListPage'
    import {columns, searchFormSchema} from './address.data';
    import {list, positionList} from './address.api';

    const {prefixCls} = useDesign('address-list');
    provide('prefixCls', prefixCls);

    // 给子组件定义一个ref变量
    const leftTree = ref();

    // 当前选中的部门code
    const orgCode = ref('');
    const positionInfo = ref({});

    // 列表页面公共参数、方法
    const {tableContext} = useListPage({
        tableProps: {
            api: list,
            columns,
            rowKey: 'id',
            showIndexColumn: true,
            formConfig: {
                labelWidth: 200,
                schemas: searchFormSchema,
            },
            canResize: false,
            actionColumn: null,
            showTableSetting: false,
            // 请求之前对参数做处理
            beforeFetch(params) {
                params.orgCode = orgCode.value
            },
        }
    });
    //注册table数据
    const [registerTable, {reload}] = tableContext;

    // 左侧树选择后触发
    function onTreeSelect(data) {
        orgCode.value = data.orgCode;
        reload();
    }

    // 查询职务信息
    async function queryPositionInfo() {
        const result = await positionList({pageSize: 99999});
        if (result) {
            let obj = {}
            result.records.forEach(position => {
                obj[position['code']] = position['name']
            })
            positionInfo.value = obj;
        }
    }
    queryPositionInfo();
</script>

<style lang="less" scoped>
    @import "./index.less";
</style>
