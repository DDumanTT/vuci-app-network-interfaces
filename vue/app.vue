<template>
  <div>
    <vuci-form
      uci-config="interfaces"
      :key="`interfaces-${tableKey}`"
      ref="tableForm"
    >
      <vuci-typed-section type="interface" :columns="columns">
        <template #name="{ s }">
          <vuci-form-item-dummy
            style="margin-bottom: 0px"
            :uci-section="s"
            name="name"
          />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy
            style="margin-bottom: 0px"
            :uci-section="s"
            name="address"
          />
        </template>
        <template #netmask="{ s }">
          <vuci-form-item-dummy
            style="margin-bottom: 0px"
            :uci-section="s"
            name="netmask"
          />
        </template>
        <template #actions="{ s }">
          <a-space>
            <a-button type="primary" @click="handleEdit(s)">Edit</a-button>
            <a-button type="danger" @click="handleDelete(s)">Delete</a-button>
          </a-space>
        </template>
      </vuci-typed-section>
      <template #footer>
        <a-form-model
          ref="form"
          :model="form"
          :rules="rules"
          layout="inline"
          :hideRequiredMark="true"
          :style="{ display: 'flex', 'justify-content': 'center' }"
        >
          <a-form-model-item label="Interface name" prop="name">
            <a-input v-model="form.name" />
          </a-form-model-item>
          <a-form-model-item>
            <a-button type="primary" @click="handleCreate">Create</a-button>
          </a-form-model-item>
        </a-form-model>
      </template>
    </vuci-form>
    <interface-modal
      :key="modalKey"
      :name="selectedInterface"
      :visible="modalVisible"
      :sid="newSectionName"
      @cancel="modalVisible = false"
      @submit="handleSubmit"
    />
  </div>
</template>

<script>
import InterfaceModal from './components/InterfaceModal.vue'

export default {
  components: { InterfaceModal },
  data () {
    return {
      columns: [
        {
          name: 'name',
          label: 'Interface name'
        },
        {
          name: 'address',
          label: 'Address'
        },
        {
          name: 'netmask',
          label: 'Netmask'
        },
        { name: 'actions' }
      ],
      modalVisible: false,
      form: {
        name: ''
      },
      rules: {
        name: [
          {
            required: true,
            message: 'Please enter an interface name',
            trigger: 'blur'
          },
          {
            max: 15,
            message: 'Interface name is too long (15 max.)',
            trigger: 'blur'
          }
        ]
      },
      selectedInterface: '',
      newSectionName: '',
      tableKey: 0,
      modalKey: 0
    }
  },
  computed: {},
  methods: {
    handleCreate () {
      this.$refs.form.validate(async (valid) => {
        if (!valid) return
        this.$uci.reset()
        const sid = this.$uci.add('interfaces', 'interface')
        this.$uci.set('interfaces', sid, 'name', this.form.name)
        await this.$uci.save()
        await this.$uci.apply()
        this.reloadTable()
        this.$refs.form.resetFields()
      })
    },
    handleSubmit () {
      this.reloadTable()
      this.modalVisible = false
    },
    handleEdit (s) {
      this.reloadModal()
      this.selectedInterface = s.name
      this.newSectionName = s['.name']
      this.modalVisible = true
    },
    handleDelete (s) {
      this.$confirm({
        title: `Delete interface ${s.name}?`,
        onOk: async () => {
          await this.$uci.del('interfaces', s['.name'])
          await this.$uci.save()
          await this.reloadTable()
        }
      })
    },
    async reloadTable () {
      this.tableKey += 1
    },
    reloadModal () {
      this.modalKey += 1
    }
  }
}
</script>
