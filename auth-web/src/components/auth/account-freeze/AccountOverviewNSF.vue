<template>
  <div>
    <p class="mb-10 subtitle">
      <v-icon
        color="#d3272c"
      >
        mdi-alert
      </v-icon>
      We were unable to process the payment from your bank account, and as a result, your account has been suspended.
      Returned system error message: {{ reason }}.
    </p>

    <v-card
      outlined
      flat
      class="suspended-info-card mb-12"
    >
      <v-card-text class="py-2 px-6">
        <v-row>
          <v-col cols="9">
            Suspended from
          </v-col>
          <v-col class="text-end">
            {{ suspendedDate }}
          </v-col>
        </v-row>
        <v-divider class="my-2" />
        <v-row>
          <v-col cols="10">
            <div>Dishonored Bank Instrument Fee</div>
            <div class="font-italic">
              As per PAD agreement, you are charged $30 dishonored bank fee for every failed payment
            </div>
          </v-col>
          <v-col class="text-end">
            ${{ nsfAmount.toFixed(2) }}
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="10">
            Total Transactions
          </v-col>
          <v-col class="text-end">
            ${{ totalAmount.toFixed(2) }}
          </v-col>
        </v-row>
        <v-divider class="my-2" />
        <v-row class="font-weight-bold">
          <v-col cols="10">
            Total Amount Due
          </v-col>
          <v-col class="text-end">
            ${{ totalAmountRemaining.toFixed(2) }}
          </v-col>
        </v-row>
      </v-card-text>
    </v-card>

    <v-divider />
    <v-row>
      <v-col
        cols="12"
        class="mt-5 pb-0 d-inline-flex"
      >
        <v-btn
          large
          text
          color="primary"
          @click="downloadNSFInvoicesPDF"
        >
          <v-icon class="ml-n2">
            mdi-download
          </v-icon>
          <span>Download Transaction Invoice (PDF)</span>
        </v-btn>
        <v-spacer />
        <v-btn
          large
          color="primary"
          @click="goNext"
        >
          <span>Next</span>
          <v-icon class="ml-2">
            mdi-arrow-right
          </v-icon>
        </v-btn>
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, onMounted, reactive, toRefs } from '@vue/composition-api'
import CommonUtils from '@/util/common-util'
import { FailedInvoice } from '@/models/invoice'
import { useOrgStore } from '@/stores/org'

export default defineComponent({
  name: 'AccountOverviewNSFView',
  emits: ['step-forward'],
  setup (_, { emit }) {
    const orgStore = useOrgStore()
    const currentOrganization = computed(() => orgStore.currentOrganization)
    const currentMembership = computed(() => orgStore.currentMembership)
    const calculateFailedInvoices: any = orgStore.calculateFailedInvoices
    const downloadNSFInvoicesPDF: any = orgStore.downloadNSFInvoicesPDF

    const state = reactive({
      nsfAmount: 0,
      totalAmount: 0,
      totalAmountRemaining: 0,
      totalPaidAmount: 0,
      reason: '',
      suspendedDate: currentOrganization.value?.suspendedOn
        ? CommonUtils.formatDateToHumanReadable(currentOrganization.value.suspendedOn) : ''
    })

    const goNext = () => {
      emit('step-forward')
    }

    onMounted(async () => {
      const failedInvoices: FailedInvoice = await calculateFailedInvoices()
      state.totalAmount = failedInvoices?.totalTransactionAmount || 0
      state.nsfAmount = failedInvoices?.nsfFee || 0
      state.totalAmountRemaining = failedInvoices?.totalAmountToPay || 0
      state.reason = failedInvoices.reason || ''
    })

    return {
      ...toRefs(state),
      currentOrganization,
      currentMembership,
      downloadNSFInvoicesPDF,
      goNext
    }
  }
})

</script>

<style lang="scss" scoped>
.subtitle {
  font-size: 1rem;
  font-weight: 500;
  color: $BCgovInputError;
}

.suspended-info-card {
  border-color: $BCgovInputError !important;
  border-width: 2px !important;

  .sub-txt {
    font-size: .75rem;
  }
}
</style>
