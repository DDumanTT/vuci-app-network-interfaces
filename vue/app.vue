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
        <a-space size="large" style="width: 100%; justify-content: center">
          <span>Interface name</span>
          <a-input v-model="interfaceName" />
          <a-button type="primary" @click="openModal">Create</a-button>
        </a-space>
      </template>
    </vuci-form>

    <a-modal
      :title="`Interface ${selectedInterface}`"
      v-model="modalVisible"
      :footer="null"
    >
      <vuci-form
        uci-config="interfaces"
        @applied="handleSubmit"
        ref="modalForm"
        :key="`modal-${modalKey}`"
      >
        <vuci-named-section
          :collapsible="false"
          :name="newSectionName"
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
          <a-button type="danger" @click="modalVisible = false"
            >Cancel</a-button
          >
        </template>
      </vuci-form>
    </a-modal>
  </div>
</template>

<script>
export default {
  data() {
    return {
      columns: [
        {
          name: "name",
          label: "Interface name",
        },
        {
          name: "address",
          label: "Address",
        },
        {
          name: "netmask",
          label: "Netmask",
        },
        { name: "actions" },
      ],
      modalVisible: false,
      interfaceName: "",
      selectedInterface: "",
      protocols: [
        ["static", "Static"],
        ["dhcp", "DHCP"],
      ],
      newSectionName: "",
      selectedProtocol: "static",
      tableKey: 0,
      modalKey: 0,
    };
  },
  computed: {},
  methods: {
    openModal() {
      if (!this.interfaceName) return;
      this.reloadModal();
      this.selectedInterface = this.interfaceName;
      this.newSectionName = this.$uci.add("interfaces", "interface");
      this.modalVisible = true;
    },
    handleProtocolSelect(e) {
      this.selectedProtocol = e.vuciSection.get(
        this.newSectionName,
        "protocol"
      );
    },
    async handleSubmit() {
      const valid = await this.$refs.modalForm.validate();
      if (!valid) return;
      this.$spin();
      if (
        this.$refs.modalInputs.get(this.newSectionName, "protocol") === "dhcp"
      ) {
        this.$uci.del("interfaces", this.newSectionName);
        await this.$uci.save();
        await this.$uci.apply();
        this.newSectionName = this.$uci.add("interfaces", "interface");
      }
      this.$uci.set(
        "interfaces",
        this.newSectionName,
        "name",
        this.interfaceName
      );
      this.$uci.set(
        "interfaces",
        this.newSectionName,
        "protocol",
        this.$refs.modalInputs.get(this.newSectionName, "protocol")
      );
      await this.$refs.modalForm.save();
      await this.$uci.save();
      await this.$uci.apply();
      this.reloadTable();
      this.reloadModal();
      this.interfaceName = "";
      this.newSectionName = "";
      this.$spin(false);
      this.modalVisible = false;
    },
    handleEdit(s) {
      this.reloadModal();
      this.selectedInterface = s.name;
      this.newSectionName = s[".name"];
      this.modalVisible = true;
    },
    handleDelete(s) {
      this.$confirm({
        title: `Delete interface ${s.name}?`,
        onOk: async () => {
          await this.$uci.del("interfaces", s[".name"]);
          await this.$uci.save();
          await this.reloadTable();
        },
      });
    },
    async reloadTable() {
      this.tableKey += 1;
    },
    reloadModal() {
      if (this.$refs.modalForm) {
        this.$refs.modalForm.reset();
        this.$refs.modalForm.load();
      }
    },
  },
};
</script>
