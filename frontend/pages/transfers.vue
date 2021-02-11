<template>
  <div>
    <section>
      <b-container class="main py-5">
        <h1 class="mb-4">
          {{ $t('pages.transfers.title') }}
        </h1>
        <div class="last-transfers">
          <div v-if="loading" class="text-center py-4">
            <Loading />
          </div>
          <template v-else>
            <!-- Filter -->
            <b-row style="margin-bottom: 1rem">
              <b-col cols="12">
                <b-form-input
                  id="filterInput"
                  v-model="filter"
                  type="search"
                  :placeholder="$t('components.transfers.search')"
                />
              </b-col>
            </b-row>
            <div class="table-responsive">
              <b-table
                striped
                hover
                :fields="fields"
                :per-page="perPage"
                :current-page="currentPage"
                :items="transfers"
                :filter="filter"
                @filtered="onFiltered"
              >
                <template #cell(block_number)="data">
                  <p class="mb-0">
                    <nuxt-link
                      v-b-tooltip.hover
                      :to="`/block?blockNumber=${data.item.block_number}`"
                      title="Check block information"
                    >
                      #{{ formatNumber(data.item.block_number) }}
                    </nuxt-link>
                  </p>
                </template>
                <template #cell(hash)="data">
                  <p class="mb-0">
                    {{ shortHash(data.item.hash) }}
                  </p>
                </template>
                <template #cell(from)="data">
                  <p class="mb-0">
                    <nuxt-link
                      :to="`/account/${data.item.from}`"
                      :title="$t('pages.accounts.account_details')"
                    >
                      <Identicon
                        :key="data.item.from"
                        :address="data.item.from"
                        :size="20"
                      />
                      {{ shortAddress(data.item.from) }}
                    </nuxt-link>
                  </p>
                </template>
                <template #cell(to)="data">
                  <p class="mb-0">
                    <nuxt-link
                      :to="`/account/${data.item.to}`"
                      :title="$t('pages.accounts.account_details')"
                    >
                      <Identicon
                        :key="data.item.to"
                        :address="data.item.to"
                        :size="20"
                      />
                      {{ shortAddress(data.item.to) }}
                    </nuxt-link>
                  </p>
                </template>
                <template #cell(amount)="data">
                  <p class="mb-0">
                    {{ formatAmount(data.item.amount) }}
                  </p>
                </template>
                <template #cell(success)="data">
                  <p class="mb-0">
                    <i
                      v-if="data.item.success"
                      class="fa fa-check-circle text-success"
                      aria-hidden="true"
                    ></i>
                    <i
                      v-else
                      class="fa fa-check-circle text-danger"
                      aria-hidden="true"
                    ></i>
                  </p>
                </template>
              </b-table>
              <div class="mt-4 d-flex">
                <b-pagination
                  v-model="currentPage"
                  :total-rows="totalRows"
                  :per-page="perPage"
                  aria-controls="validators-table"
                />
                <b-button-group class="ml-2">
                  <b-button
                    v-for="(item, index) in tableOptions"
                    :key="index"
                    @click="handleNumFields(item)"
                  >
                    {{ item }}
                  </b-button>
                </b-button-group>
              </div>
            </div>
          </template>
        </div>
      </b-container>
    </section>
  </div>
</template>

<script>
import gql from 'graphql-tag'
import commonMixin from '@/mixins/commonMixin.js'
import Loading from '@/components/Loading.vue'
import Identicon from '@/components/Identicon.vue'
import { paginationOptions } from '@/frontend.config.js'

export default {
  components: {
    Identicon,
    Loading,
  },
  mixins: [commonMixin],
  data() {
    return {
      loading: true,
      filter: null,
      filterOn: [],
      transfers: [],
      tableOptions: paginationOptions,
      perPage: localStorage.paginationOptions
        ? parseInt(localStorage.paginationOptions)
        : 10,
      currentPage: 1,
      totalRows: 1,
      fields: [
        {
          key: 'block_number',
          label: 'Block',
          sortable: true,
        },
        {
          key: 'hash',
          label: 'Hash',
          sortable: true,
        },
        {
          key: 'from',
          label: 'From',
          sortable: true,
        },
        {
          key: 'to',
          label: 'To',
          sortable: true,
        },
        {
          key: 'amount',
          label: 'Amount',
          sortable: true,
        },
        {
          key: 'success',
          label: 'Success',
          sortable: true,
        },
      ],
    }
  },
  head() {
    return {
      title: 'Explorer | Paralink Network',
      meta: [
        {
          hid: 'description',
          name: 'description',
          content:
            'Paralink block explorer. Paralink is a multi-chain oracle platform for DeFi applications',
        },
      ],
    }
  },
  methods: {
    handleNumFields(num) {
      localStorage.paginationOptions = num
      this.perPage = parseInt(num)
    },
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
    },
  },
  apollo: {
    $subscribe: {
      extrinsic: {
        query: gql`
          subscription extrinsic {
            extrinsic(
              where: {
                section: { _eq: "balances" }
                method: { _like: "transfer%" }
              }
              order_by: { block_number: desc, extrinsic_index: desc }
            ) {
              block_number
              signer
              hash
              args
              success
            }
          }
        `,
        result({ data }) {
          this.transfers = data.extrinsic.map((transfer) => {
            return {
              block_number: transfer.block_number,
              hash: transfer.hash,
              from: transfer.signer,
              to: JSON.parse(transfer.args)[0],
              amount: JSON.parse(transfer.args)[1],
              success: transfer.success,
            }
          })
          this.totalRows = this.transfers.length
          this.loading = false
        },
      },
    },
  },
}
</script>
