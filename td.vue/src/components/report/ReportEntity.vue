<template>
    <b-col class="td-threat-data no-print">
        <b-row>
            <b-col>
                <h3 class="entity-title">
                    {{ `${entity.data.name} (${dataType})` }}
                    <em v-if="outOfScope">- {{ $t('threatmodel.properties.outOfScope') }}</em>
                </h3>
            </b-col>
        </b-row>
        <b-row v-if="entity.data.description">
            <b-col>
                <h4>Description</h4>
                <p class="entity-description">{{ entity.data.description }}</p>
            </b-col>
        </b-row>
        <b-row v-if="showProperties && propertiesData">
            <b-col md="12">
                <h4>Properties</h4>
                <b-table
                    :fields="propertiesFields"
                    :items="propertiesData"
                    striped
                    responsive>
                </b-table>
            </b-col>
        </b-row>
        <b-row v-if="tableData.length > 0">
            <b-col md="12">
                <h4>Threats</h4>
                <b-table
                    :data-test-id="entity.data.name.replace(' ', '_')"
                    :items="tableData"
                    striped
                    responsive>
                </b-table>
            </b-col>
        </b-row>
    </b-col>
</template>

<style lang="scss" scoped>
.td-threat-data {
    white-space: pre-wrap;
}

.entity-title {
    white-space: normal;
}
</style>

<script>
import threatService from '@/service/threats/index.js';

export default {
    name: 'TdReportEntity',
    props: {
        entity: Object,
        outOfScope: {
            type: Boolean,
            default: false
        },
        showOutOfScope: {
            type: Boolean,
            default: true
        },
        showMitigated: {
            type: Boolean,
            default: true
        },
        showProperties: {
            type: Boolean,
            default: true
        }
    },
    data() {
        return {
            propertiesFields: [
                {key: "name", label: this.$t("report.name")},
                {key: "value", label: this.$t("report.value")}
            ]
        };
    },
    computed: {
        dataType: function () {
            const entityType = this.entity.data.type.replace('tm.', '').replace('td.', '');
            return this.$t(`threatmodel.shapes.${this.toCamelCase(entityType)}`);
        },
        tableData: function () {
            return threatService.filterForDiagram(this.entity.data, {
                showOutOfScope: this.showOutOfScope,
                showMitigated: this.showMitigated
            }).map((threat) => {
                return {
                    [this.$t('threats.properties.number')]: threat.number,
                    [this.$t('threats.properties.title')]: threat.title,
                    [this.$t('threats.properties.type')]: threat.type,
                    [this.$t('threats.properties.priority')]: threat.severity,
                    [this.$t('threats.properties.status')]: threat.status,
                    [this.$t('threats.properties.score')]: threat.score,
                    [this.$t('threats.properties.description')]: threat.description,
                    [this.$t('threats.properties.mitigation')]: threat.mitigation

                };
            });
        },
        propertiesData: function () {
            const boolToString = (b) => b ? this.$t("report.yes") : this.$t("report.no");
            switch (this.entity.data.type) {
                case "tm.Flow":
                    return [
                        {name: this.$t("threatmodel.properties.protocol"), value: this.entity.data.protocol},
                        {name: this.$t("threatmodel.properties.isEncrypted"), value: boolToString(this.entity.data.isEncrypted)},
                        {name: this.$t("threatmodel.properties.publicNetwork"), value: boolToString(this.entity.data.isPublicNetwork)},
                    ];
                case "tm.Process":
                    return [
                        {name: this.$t("threatmodel.properties.handlesCardPayment"), value: boolToString(this.entity.data.handlesCardPayment)},
                        {name: this.$t("threatmodel.properties.handlesGoodsOrServices"), value: boolToString(this.entity.data.handlesGoodsOrServices)},
                        {name: this.$t("threatmodel.properties.isWebApplication"), value: boolToString(this.entity.data.isWebApplication)},
                    ];
                case "tm.Store":
                    return [
                        {name: this.$t("threatmodel.properties.isALog"), value: boolToString(this.entity.data.isALog)},
                        {name: this.$t("threatmodel.properties.isEncrypted"), value: boolToString(this.entity.data.isEncrypted)},
                        {name: this.$t("threatmodel.properties.storesInventory"), value: boolToString(this.entity.data.storesInventory)},
                        {name: this.$t("threatmodel.properties.storesCredentials"), value: boolToString(this.entity.data.storesCredentials)},
                        {name: this.$t("threatmodel.properties.isSigned"), value: boolToString(this.entity.data.isSigned)},
                    ];
            }
        }
    },
    methods: {
        toCamelCase(str) {
            // https://stackoverflow.com/questions/2970525/converting-any-string-into-camel-case
            return str.replace(/(?:^\w|[A-Z]|\b\w)/g, (ltr, idx) => idx === 0 ? ltr.toLowerCase() : ltr.toUpperCase()).replace(/\s+/g, '');
        }
    }
};

</script>
