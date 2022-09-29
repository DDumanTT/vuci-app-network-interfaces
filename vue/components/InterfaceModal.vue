<template>
  <a-modal
    :title="`Interface ${name}`"
    :visible="visible"
    :footer="null"
    @cancel="$emit('cancel')"
  >
    <vuci-form
      uci-config="interfaces"
      ref="modalForm"
    >
      <vuci-named-section
        :collapsible="false"
        :name="sid"
        type="interface"
        label="Interface"
        :card="false"
        ref="modalInputs"
        v-slot="{ s }"
      >
        <vuci-form-item-select
          :uci-section="s"
          :initial="selectedProtocol"
          :options="protocols"
          name="protocol"
          label="Protocol"
          @change="handleProtocolSelect"
          required
        />
        <vuci-form-item-input
          :uci-section="s"
          name="address"
          label="Address"
          rules="ip4addr"
          depend="protocol === 'static'"
          required
        />
        <vuci-form-item-input
          :uci-section="s"
          name="netmask"
          label="Netmask"
          rules="netmask4"
          depend="protocol === 'static'"
          required
        />
        <vuci-form-item-input
          :uci-section="s"
          name="gateway"
          label="Gateway"
          rules="ip4addr"
          depend="protocol === 'static'"
          required
        />
        <vuci-form-item-list
          :uci-section="s"
          name="dns"
          label="DNS"
          rules="ip4addr"
          depend="protocol === 'static'"
          required
        />
      </vuci-named-section>
      <template #footer>
        <a-button type="primary" @click="handleSubmit">Save</a-button>
        <a-button type="danger" @click="$emit('cancel')">Cancel</a-button>
      </template>
    </vuci-form>
  </a-modal>
</template>

<script>
export default {
  data () {
    return {
      protocols: [
        ['static', 'Static'],
        ['dhcp', 'DHCP']
      ],
      selectedProtocol: 'static'
    }
  },
  props: ['name', 'visible', 'sid'],
  methods: {
    async handleSubmit () {
      const valid = await this.$refs.modalForm.validate()
      if (!valid) return
      this.$spin()
      let sid = this.sid
      if (this.$refs.modalInputs.get(this.sid, 'protocol') === 'dhcp') {
        this.$uci.del('interfaces', sid)
        await this.$uci.save()
        await this.$uci.apply()
        sid = this.$uci.add('interfaces', 'interface')
      }
      this.$uci.set(
        'interfaces',
        sid,
        'name',
        this.name
      )
      this.$uci.set(
        'interfaces',
        sid,
        'protocol',
        this.$refs.modalInputs.get(this.sid, 'protocol')
      )
      await this.$refs.modalForm.save()
      await this.$uci.save()
      await this.$uci.apply()
      this.$spin(false)
      this.$emit('submit')
    },
    handleProtocolSelect (e) {
      this.selectedProtocol = e.vuciSection.get(
        this.sid,
        'protocol'
      )
    }
  }
}
</script>
