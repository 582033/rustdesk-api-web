<template>
  <div>
    <a-card class="list-query" :bordered="false">
      <a-form layout="inline" :model="listQuery">
        <a-form-item :label="T('Username')">
          <a-input v-model:value="listQuery.username" />
        </a-form-item>
        <a-form-item>
          <a-button type="primary" @click="handlerQuery">{{ T('Filter') }}</a-button>
          <a-button type="primary" @click="toAdd" style="margin-left: 8px;">{{ T('Add') }}</a-button>
          <a-button type="primary" @click="toExport" style="margin-left: 8px;">{{ T('Export') }}</a-button>
        </a-form-item>
      </a-form>
    </a-card>
    <a-card class="list-body" :bordered="false">
      <a-table :data-source="listRes.list" :loading="listRes.loading" :columns="columns" bordered :pagination="false" rowKey="id">
        <template #bodyCell="{ column, record }">
          <template v-if="column.key === 'group_id'">
            <span v-if="record.group_id"> <a-tag>{{ listRes.groups?.find(g => g.id === record.group_id)?.name }} </a-tag> </span>
            <span v-else> - </span>
          </template>
          <template v-if="column.key === 'status'">
            <a-switch :checked="record.status === ENABLE_STATUS" @change="checked => changeStatus(record, checked)" />
          </template>
          <template v-if="column.key === 'actions'">
            <div class="actions-cell">
              <a-button size="small" @click="toTag(record)">{{ T('UserTags') }}</a-button>
              <a-button size="small" @click="toAddressBook(record)">{{ T('UserAddressBook') }}</a-button>
              <a-button size="small" @click="toEdit(record)">{{ T('Edit') }}</a-button>
              <a-button type="dashed" size="small" @click="copyCredentials(record)">{{ T('Copy') }}</a-button>
              <a-button type="dashed" danger size="small" @click="changePass(record)">{{ T('ResetPassword') }}</a-button>
              <a-button type="primary" danger size="small" @click="remove(record)">{{ T('Delete') }}</a-button>
            </div>
          </template>
        </template>
      </a-table>
      <a-pagination
          style="margin-top: 12px; text-align: right;"
          v-model:current="listQuery.page"
          v-model:pageSize="listQuery.page_size"
          :total="listRes.total"
          show-size-changer
          show-quick-jumper
          :show-total="total => `${T('Total')} ${total} ${T('Items')}`"
      />
    </a-card>
  </div>
</template>

<script setup>
  import { handleClipboard } from '@/utils/clipboard'
  import { useRepositories, useDel, useToEditOrAdd, useChangePwd } from '@/views/user/composables'
  import { T } from '@/utils/i18n'
  import { DISABLE_STATUS, ENABLE_STATUS } from '@/utils/common_options'
  import { update } from '@/api/user'
  import { message } from 'ant-design-vue'
  import { onMounted, watch, computed } from 'vue'

  const {
    listRes, listQuery, handlerQuery, getList, getGroups, toExport,
  } = useRepositories()

  const columns = computed(() => [
    { title: 'ID', dataIndex: 'id', key: 'id', align: 'center' },
    { title: T('Username'), dataIndex: 'username', key: 'username', align: 'center' },
    { title: T('Email'), dataIndex: 'email', key: 'email', align: 'center' },
    { title: T('Nickname'), dataIndex: 'nickname', key: 'nickname', align: 'center' },
    { title: T('Group'), key: 'group_id', align: 'center' },
    { title: T('Status'), key: 'status', align: 'center' },
    { title: T('Remark'), dataIndex: 'remark', key: 'remark', align: 'center' },
    { title: T('CreatedAt'), dataIndex: 'created_at', key: 'created_at', align: 'center' },
    { title: T('UpdatedAt'), dataIndex: 'updated_at', key: 'updated_at', align: 'center' },
    { title: T('Actions'), key: 'actions', align: 'center', width: 650 },
  ]);

  onMounted(getGroups)
  onMounted(getList)

  watch(() => listQuery.page, getList)
  watch(() => listQuery.page_size, handlerQuery)

  const { toEdit, toAdd, toAddressBook, toTag } = useToEditOrAdd()
  const { changePass } = useChangePwd()
  const { del } = useDel()
  const remove = async (row) => {
    const res = await del(row.id)
    if (res) {
      getList(listQuery)
    }
  }

  const copyCredentials = async (row) => {
    const username = row.username
    const password = btoa(username + '\n')
    const textToCopy = `您的用户名为 ${username} 密码为 ${password}`
    try {
      await handleClipboard(textToCopy)
      message.success('已成功复制到剪贴板')
    } catch (err) {
      message.error('复制失败，请稍后重试')
      console.error('Clipboard write failed: ', err)
    }
  }

  const changeStatus = async (row, checked) => {
    const newStatus = checked ? ENABLE_STATUS : DISABLE_STATUS;
    const res = await update({ ...row, status: newStatus }).catch(_ => false)
    if (res) {
      message.success(T('OperationSuccess'))
    }
    // No matter success or fail, refresh the list to get the real status
    getList(listQuery)
  }
</script>

<style scoped>
.actions-cell {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
}
</style>