<template>
  <table class="portfolio_item">
    <thead>
    <template v-if="item">
      <tr v-if="coin">
        <th :class="['portfolio_item-cell', 'portfolio_item-name', `portfolio_item-name--${coin.ticker}`]">
          <small :class="[`portfolio_item-name--pseudo`]" :style="{backgroundColor: coin.color}"></small>
          <div class="portfolio_item-logo">
            <img :src="coin.icon"
                 :alt="coin.title">
          </div>
          <span>{{ coin.title }} · <b>{{ coin.ticker }}</b></span></th>

        <th class="portfolio_item-cell portfolio_item-amount">
          <div class="portfolio_item-amount-wrapper">
            <span>{{ item.amount }} <b>{{ coin.ticker }}</b></span>
            <a href="#" class="portfolio_item-edit" data-toggle="popover" data-placement="right" data-original-title="" title=""></a>
          </div>
        </th>

        <th class="portfolio_item-cell portfolio_item-percent">
          <span v-if="totalPercent"> {{ totalPercent }}%</span>
        </th>

        <th class="portfolio_item-cell portfolio_item-price">
          <span class="portfolio_item-price-usd" v-html="currencyConverter(countValue(item.amount , item.id), currentCurrency.ticker )"></span>
          <span class="portfolio_item-price-btc" v-html="currencyConvertUSDToBTC(countValue(item.amount , item.id))"></span>
        </th>

        <th :class="['portfolio_item-cell', 'portfolio_item-24hour', `portfolio_item-24hour--${ isPositive (coin) }`]">
          <span v-html=" coin['24_change']+'%'"></span>
        </th>

        <th class="portfolio_item-cell portfolio_item-capitalizations">
          <span v-html=" currencyConverter(coin.market_cap, currentCurrency.ticker ) "></span>
        </th>

        <th class="portfolio_item-cell portfolio_item-volume" v-if="coin">
          <div class="portfolio_item-volume-wrapper">
            <span v-html=" currencyConverter(coin['24H_volume'], currentCurrency.ticker ) "></span>
            <a href="#" @click.prevent="localCollapse()" class="portfolio_item-dropdown js-thead-show"></a>
          </div>
        </th>
      </tr>
    </template>
    </thead>

    <tbody :id="`coin-toggle--${coin.ticker}`"
           v-if="coin" :class="['tbody', `tbody--${coin.ticker}`, 'js-tbody', {collapsed: !collapseOpen}]">
    <tr>
      <td>
        <span>{{ $t('portfolio.Operation') }}</span>
      </td>
      <td>
        <span>{{ $t('portfolio.Quantity') }}</span>
      </td>
      <td colspan="2">
        <span>{{ $t('portfolio.Price') }}</span>
      </td>
      <td colspan="3">
        <span>{{ $t('portfolio.Price_per_coin') }}</span>
        <a href="#" :id="`removeCoinForm-${coin.id}`" @click.prevent="onEnable(`removeCoinForm-${coin.id}`)" class="removeCoin"></a>

        <b-popover :target="`removeCoinForm-${coin.id}`"
                   :placement="'top'"
                   triggers="hover"
        >
              <span v-bind:style="[
                           {padding: '8px'},
                           {fontSize: '12px'},
                           {textAlign: 'center'},
                           {backgroundColor: '#ffffff'},
                           {boxShadow: '-8px 8px 24px rgba(0, 0, 0, 0.2)'}
                            ]">{{ $t('portfolio.remove_coin_short') }}</span>
        </b-popover>

        <popover-coin-delete :id="coin.id"
                             :coin="coin"
                             v-on:confirm="removeCoin()"
                             v-on:close="onDisable(`removeCoinForm-${coin.id}`)"></popover-coin-delete>
      </td>
    </tr>
    <small :class="[`tbody--pseudo`]" :style="{backgroundColor: coin.color}"></small>
    <template v-if="item" v-for="(operation, index) in item.operations">
      <tr :key="index">

        <template v-if="operation.type === 'swapped'">
          <td>
            <span :class="[ operationType(operation) ]">{{ $t(`portfolio.${operationType(operation) }`) }}</span>
          </td>
          <td>
            <span>{{ operation.quantity }} <b>{{ coin.ticker }}</b></span>
            <span class="swapped_coin"><small>{{ $t(`portfolio.IN`)}}</small> {{ countSwappedCoinQuantity(operation) }} <b>{{ swappedCoinTicker(operation) }}</b></span>
          </td>
          <td colspan="2">
            <span v-html="currencyConverter(operation.price, currentCurrency.ticker )"></span>
          </td>
          <td colspan="2">
            <span v-html="currencyConverter( operation.price_per_coin, currentCurrency.ticker )"></span>
            <span class="swapped_coin" v-html="pricePerCoinSwapped(operation)"></span>
          </td>
        </template>

        <template v-else>
          <td>
            <span :class="[ operationType(operation) ]">{{ $t(`portfolio.${operationType(operation) }`) }}</span>
          </td>
          <td>
            <span>{{ operation.quantity }} <b>{{ coin.ticker }}</b></span>
          </td>
          <td colspan="2">
            <span v-html="currencyConverter(countOperationValue(operation), currentCurrency.ticker )"></span>
          </td>
          <td colspan="2">
            <span v-html="currencyConverter( operation.price_per_coin, currentCurrency.ticker )"></span>
          </td>
        </template>
        <td>
          <!--<a href="#" :id="`editConfirmForm-${index}-${item.id}`" @click.prevent="onEnable(`editConfirmForm-${index}-${item.id}`)" class="edit">{{ $t(`form.edit`) }}</a>-->
          <button @click="toggleEditOperation(`formEditRecord--${index}${item.id}`)" class="edit">{{ $t(`form.edit`) }}</button>
          <button :id="`deleteConfirmForm-${index}-${item.id}`" @click="onEnable(`deleteConfirmForm-${index}-${item.id}`)" class="delete">{{ $t(`form.delete`) }}</button>
          <span>

                <popover-operation-delete :index="index"
                                          :id="item.id"
                                          v-on:confirm="deleteOperation(operation, index)"
                                          v-on:close="onDisable(`deleteConfirmForm-${index}-${item.id}`)"></popover-operation-delete>

            <!--<popover-operation-edit :index="index"-->
            <!--:coinId="item.id"-->
            <!--:operation="operation"-->
            <!--v-on:confirm="editOperation(index, payload = $event)"-->
            <!--v-on:close="onDisable(`editConfirmForm-${index}-${item.id}`)"></popover-operation-edit>-->
            </span>
        </td>
      </tr>
      <tr :key="index+item.operations.length">
        <td class="form_td" colspan="12">
          <b-collapse :id="`formEditRecord--${index}${item.id}`">
            <form-edit-operation-portfolio :index="index"
                                           :selectedCoin="item"
                                           :portfolio="portfolio"
                                           :operation="operation"></form-edit-operation-portfolio>
          </b-collapse>
        </td>
      </tr>
    </template>
    <tr>
      <td colspan="12">

        <form-add-record-portfolio :selectedCoin="item" :portfolio="portfolio"></form-add-record-portfolio>

        <!-- /.portfolio_form -->
      </td>
    </tr>
    <tr>
      <td class="exchange" colspan="12">
        <PortfolioExchangeMarkets :coin="item" ></PortfolioExchangeMarkets>
      </td>
    </tr>
    </tbody>

  </table>
  <!-- /.portfolio_item -->
</template>

<script>
import Vuex from 'vuex'
import { EventBus } from './eventbus'
import formAddRecordPortfolio from './formAddRecordPortfolio'
import formEditOperationPortfolio from './formEditOperationPortfolio'
import popoverOperationDelete from './popoverOperationDelete'
import popoverOperationEdit from './popoverOperationEditPortfolio'
import popoverCoinDelete from './popoverCoinDelete'
import PortfolioExchangeMarkets from './PortfolioExchangeMarkets'
const storeEvent = Vuex.createNamespacedHelpers('portfolio')
export default {
  name: 'PortfolioListItem',
  components: { formAddRecordPortfolio, popoverOperationDelete, popoverOperationEdit, popoverCoinDelete, PortfolioExchangeMarkets, formEditOperationPortfolio },
  props: {
    portfolio: Object,
    item: Object,
    totalPercent: String
  },
  data () {
    return {
      collapseOpen: false
    }
  },
  computed: {
    ...storeEvent.mapGetters({
      getCoinByID: 'getCoinByID',
      currencyConverter: 'currencyConverter',
      currencyConvertUSDToBTC: 'currencyConvertUSDToBTC',
      currentCurrency: 'currentCurrency'
    }),
    coin: function () {
      if (this.getCoinByID(this.item.id)) { return this.getCoinByID(this.item.id) }
    }
  },
  watch: {
    /**
       * Смотрим за объектом portfolio и если он изменился,
       * то закрываем все раскрытые меню и обнуляем донут чарт
       * */
    portfolio (newVal, oldVal) {
      if (newVal.id !== oldVal.id) {
        this.collapseOpen = false
        EventBus.$emit('chart:doughnut:check', 'All', false)
      }
    }
  },
  methods: {
    /**
       * Открыть форму изменения операции
       * */
    toggleEditOperation (id) {
      this.$root.$emit('bv::toggle::collapse', id)
    },
    /**
       * Удалить монету
       * */
    removeCoin () {
      let portfolioChanged = JSON.parse(JSON.stringify(this.portfolio))

      portfolioChanged.coin = portfolioChanged.coin.filter(coin => {
        if (coin.id !== this.item.id) {
          return coin
        }
      })
      this.onDisable(`removeCoinForm-${this.coin.id}`)
      this.collapseOpen = !this.collapseOpen
      this.$store.dispatch('portfolio/changePortfolio', portfolioChanged)
    },
    /**
       * Показать поповер
       * */
    onEnable (index) {
      this.$root.$emit('bv::show::popover', index)
    },
    /**
       * Скрыть поповер
       * */
    onDisable (index) {
      this.$root.$emit('bv::hide::popover', index)
    },
    /**
       * Удалить операцию
       * */
    deleteOperation (operation, index) {
      let portfolioChanged = JSON.parse(JSON.stringify(this.portfolio))

      /**
         * Фильтруем объект портфолио на предмет текущей монеты
         * */
      portfolioChanged.coin.filter(coin => {
        if (coin.id === this.item.id) {
          /**
             * //TODO нужно ли высчитывать количество монет при удалении операции.
             * */
          // coin.amount = parseFloat(coin.amount) - parseFloat(coin.operations[index].quantity)
          coin.operations.splice(index, 1)
        }
        return coin
      })
      this.$store.dispatch('portfolio/changePortfolio', portfolioChanged)
      this.onDisable(`deleteConfirmForm-${index}-${this.item.id}`)
    },
    /**
       * Цена за монету, на которую меняем
       * */
    pricePerCoinSwapped (operation) {
      return `<small>${this.$t('portfolio.IN')} </small>${this.currencyConverter(operation.price_per_coin_swapped, this.currentCurrency.ticker)}`
    },
    /**
       * Тикер обменной монеты
       * */
    swappedCoinTicker (operation) {
      let coin = this.getCoinByID(operation.deal_currency)
      return coin.ticker
    },
    /**
       * Считает количество монет, полученных при обмене
       * */
    countSwappedCoinQuantity (operation) {
      let count = operation.price / operation.price_per_coin_swapped
      let decimals = 4
      return Number(Math.round(count + `e${decimals}`) + `e-${decimals}`)
    },
    operationType (operation) {
      return operation.type
    },
    isPositive (coin) {
      return coin['24_change'] > 0 ? 'positive' : 'negative'
    },
    /**
       * Посчитать стоимость. Количество на цену за еденицу
       * */
    countValue (amount, coinID) {
      if (this.getCoinByID(coinID)) {
        return (parseFloat(this.getCoinByID(coinID)['market_price']) * parseFloat(amount))
      }
    },
    /**
       * Посчитать цену операции.
       * */
    countOperationValue (operation) {
      return (parseFloat(operation.price_per_coin) * parseFloat(operation.quantity))
    },
    /**
       * Скрыть\Показать информацию о монете. Локальный триггер
       * */
    localCollapse () {
      this.collapseOpen = !this.collapseOpen
      if (this.getCoinByID(this.item.id)) {
        EventBus.$emit('chart:doughnut:check', this.getCoinByID(this.item.id)['ticker'], this.collapseOpen)
      }
    },
    /**
       * Скрыть\Показать информацию о монете. Глобальный триггер с события
       * */
    toggleCollapseCoin (e, value) {
      if (e === this.coin.ticker) {
        this.collapseOpen = value
        /**
           * Шлем в чарт сообщение о закрытии\открытии
           * */
        console.log('event>>>>>>', e, value)
        this.$root.$emit('bv::toggle::collapse', `coin-toggle--${e}`, value)
      }
    }
  },
  created () {
    /**
       * Весим событие, при котором срабатывает функция открывающая\закрывающая инфу о коине
       * */
    EventBus.$on('chart:doughnut:grow', this.toggleCollapseCoin)
  }
}
</script>

<style lang="less" scoped>
  @import "../assets/less/vars";
  body /deep/ .popover-remove-coin{
    padding: 8px;
    font-size: 12px;
    text-align: center;
    color: #525c6c;
  }
  .portfolio{
    .removeCoin{
      cursor: pointer;
      position: absolute;
      width: 11px;
      height: 13px;
      right: 29px;
      top: 24px;
      background-image: @img-trash;
      background-position: center;
      background-repeat: no-repeat;
      background-size: contain;
      z-index: 11;
    }
    &_item {
      display: table;
      width: 100%;
      table-layout: fixed;
      position: relative;
      background-color: #fafbfc;
      //height: 52px;
      &-cell {
        padding: 14px 12px;
        border-right: 1px solid #e9ecee;
        &:last-child {
          border-right: none;
        }
      }
      &:hover {
        background-color: #fff;
        box-shadow: 0 1px 0 #e0e6ed, 0 -1px 0 #e0e6ed;
      }
      &--active {
        box-shadow: 0 1px 0 #e0e6ed, 0 -1px 0 #e0e6ed;
        background-color: #fff;
      }
      &-logo {
        display: inline-block;
        margin-right: 8px;
      }
      &-name {
        width: 20%;
        position: relative;
        display: table-cell;
        padding: 0px 10px 0 24px;
        span {
          color: #333f52;
          font-family: @medium;
          font-weight: 500;
          font-size: 14px;
          line-height: 14px;
          letter-spacing: -0.2px;
        }
        b {
          font-family: @medium;
          font-weight: 500;
          color: #8f96a1;
          letter-spacing: -0.2px;
        }
        &--pseudo{
          position: absolute;
          width: 4px;
          height: 100%;
          left: 0px;
          top: 0px;
          z-index: 11;
        }
      }
      &-amount {
        width: 20%;
        display: table-cell;
        &-wrapper {
          display: flex;
          justify-content: space-between;
        }
        span {
          color: #333f52;
          font-family: @medium;
          font-weight: 500;
          font-size: 16px;
          line-height: 16px;
          text-transform: uppercase;
        }
        b {
          font-family: @medium;
          font-weight: 500;
          font-size: 12px;
        }
      }
      &-edit {
        width: 12px;
        height: 12px;
        background-image: @img-edit;
      }
      &-percent {
        width: 6%;
        display: table-cell;
        text-align: center;
        span {
          //margin: auto;
          color: #333f52;
          font-family: @medium;
          font-weight: 500;
          font-size: 16px;
          line-height: 16px;
        }
      }
      &-price {
        width: 12%;
        display: table-cell;
        padding: 14px 12px 0px 12px;
        span /deep/ b {
          font-size: 12px;
        }
        &-usd {
          display: block;
          white-space: nowrap;
          color: #333f52;
          font-family: @medium;
          font-weight: 500;
          font-size: 16px;
          line-height: 16px;
        }
        &-btc {
          opacity: 0.6;
          color: #8f96a1;
          font-family: @medium;
          font-weight: 500;
          font-size: 9px;
          line-height: 12px;
          text-transform: uppercase;
          position: relative;
          top: -3px;
        }
      }
      &-24hour {
        width: 10%;
        display: table-cell;
        b {
          padding: 2px 8px 2px;
          color: #8f96a1;
          border-radius: 9px;
          border: 1px solid #8f96a1;
          font-family: @main-font;
          font-weight: 700;
          font-size: 11px;
          line-height: 11px;
          text-transform: uppercase;
        }
        span {
          position: relative;
          margin: auto;
          padding-left: 18px;
          color: #707986;
          font-family: @main-font;
          font-weight: 400;
          font-size: 14px;
          line-height: 16px;
          &::before {
            content: "";
            position: absolute;
            width: 20px;
            height: 16px;
            left: -4px;
            top: 0px;
            background-image: @img-triangle-green;
            background-position: center;
            background-repeat: no-repeat;
            z-index: 11;
          }
        }
        &--negative {
          span {
            &::before {
              left: -4px;
              top: 0px;
              transform: rotateX(180deg);
              background-image: @img-triengle-red;
            }
          }
        }
      }
      &-capitalizations {
        width: 13%;
        display: table-cell;
        span {
          color: #707986;
          font-family: @medium;
          font-weight: 500;
          font-size: 16px;
          line-height: 16px;
          text-transform: uppercase;
        }
        b, /deep/ b {
          font-size: 12px;
          font-family: @medium;
        }
      }
      &-volume {
        width: 19%;
        display: table-cell;
        padding: 14px 22px 14px 12px;
        &-wrapper {
          display: flex;
          align-items: center;
          justify-content: space-between;
        }
        span {
          color: #707986;
          font-family: @medium;
          font-weight: 500;
          font-size: 16px;
          line-height: 16px;
          text-transform: uppercase;
        }
        b, /deep/ b {
          font-size: 12px;
        }
        a {
          width: 12px;
          height: 12px;
          background-image: @img-dropdown-portfolio;
        }
      }
    }
  }
  table {
    width: 100%;
    table-layout: fixed;
    .tbody {
      position: relative;
      background-color: #fff;
      transition: all .3s ease;
      height: 100%;
      &.collapsed{
        display: none;
      }
      &::before {
        content: "";
        position: absolute;
        //top: 48px;
        left: 0;
        width: 100%;
        height: 10px;
        background-image: linear-gradient(180deg, #e0e6ed 0%, rgba(224, 230, 237, 0) 100%);
      }
      &--pseudo{
        position: absolute;
        width: 4px;
        height: 100%;
        left: 0px;
        top: 1px;
        //background-color: #bcbdbc;
        z-index: 11;
      }
      tr {
        &:first-child {
          border-bottom: 1px solid #e9ecee;

          td {
            padding: 30px 12px 9px 12px;
            border-right: 1px solid #e9ecee;
            position: relative;
            span {
              color: #525c6c;
              font-family: @main-font;
              font-weight: 400;
              font-size: 13px;
              line-height: 12px;
            }
            &:first-child {
              padding: 30px 12px 9px 58px;
            }
            &:nth-last-child(2) {
              border-right: 1px solid #e9ecee;
            }
            &:last-child {
              text-align: left;
              border-right: none;
            }
          }
        }
        &:last-child {
          td {
            &:first-child {
              padding: 0;
              text-align: right;
            }
          }
        }
        td.form_td{
          padding: 0px 12px 0px 58px!important;
        }
        td {
          padding: 15px 24px 13px 12px;
          border-right: 1px solid #e9ecee;
          &:first-child {
            padding: 12px 12px 9px 58px;
            .bought {
              position: relative;
              padding-left: 22px;
              color: #45af37;
              &::before {
                content: "";
                position: absolute;
                width: 17px;
                height: 17px;
                left: -1px;
                top: 3px;
                background-image: @img-bought;
                background-position: center;
                background-repeat: no-repeat;
                z-index: 11;
              }
            }
            .sold {
              position: relative;
              padding-left: 22px;
              color: #ff3657;
              &::before {
                content: "";
                position: absolute;
                width: 17px;
                height: 17px;
                left: -1px;
                top: 3px;
                background-image: @img-sold;
                background-position: center;
                background-repeat: no-repeat;
                z-index: 11;
              }
            }
            .swapped {
              position: relative;
              padding-left: 22px;
              color: #1991eb;
              &::before {
                content: "";
                position: absolute;
                width: 17px;
                height: 17px;
                left: -1px;
                top: 3px;
                background-image: @img-swapped;
                background-position: center;
                background-repeat: no-repeat;
                z-index: 11;
              }
            }
          }
          &:last-child {
            text-align: right;
            border-right: none;
          }
          &:nth-last-child(2) {
            border-right: none;
          }
          span {
            color: #333f52;
            font-family: @medium;
            font-weight: 500;;
            font-size: 16px;
            line-height: 16px;
            b, /deep/ b {
              font-size: 12px;
            }
          }
          .swapped_coin{
            display: block;
            height: 12px;
            color: #707986;
            font-size: 12px;
            font-weight: 500;
            line-height: 12px;
            text-transform: uppercase;
            letter-spacing: -0.04px;
            /deep/ small{
            }
            /deep/ b{
            }
          }
          .edit {
            position: relative;
            display: inline-block;
            margin-right: 28px;
            padding-left: 15px;
            color: #45af37;
            font-family: @main-font;
            font-weight: 400;
            font-size: 12px;
            line-height: 12px;
            text-decoration: underline;
            text-decoration-style: dashed;
            border: none;
            background: none;
            outline: none;
            cursor: pointer;
            &::before {
              content: "";
              position: absolute;
              width: 12px;
              height: 12px;
              left: -1px;
              top: 0px;
              background-image: @img-edit-green;
              background-position: center;
              background-repeat: no-repeat;
              z-index: 11;
            }
          }
          .delete {
            position: relative;
            padding-left: 15px;
            color: #8f96a1;
            font-family: @main-font;
            font-weight: 400;
            font-size: 10px;
            line-height: 14px;
            border: none;
            background: none;
            outline: none;
            cursor: pointer;
            padding-right: 0px;
            &::before {
              content: "";
              position: absolute;
              width: 12px;
              height: 12px;
              left: -1px;
              top: 0px;
              background-image: @img-delete-grey;
              background-position: center;
              background-repeat: no-repeat;
              z-index: 11;
            }
          }
        }
      }

    }
    tr.t-foot {
      td {
        padding: 12px 12px 12px 12px;
        color: #707986;
        font-family: @main-font;
        font-weight: 500;
        font-size: 16px;
        line-height: 16px;
        b {
          font-size: 12px;
        }
        &:nth-child(2) {
          text-align: right;
          border-right: 1px solid #e9ecee;
        }
        &:last-child {
          text-align: right;
          padding: 12px 28px 12px 12px;
        }
      }
      &:nth-child(odd) {
        background-color: rgba(232, 237, 242, 0.44);
      }
      &:first-child {
        background-color: #fff;
        border-bottom: 1px solid #e9ecee;
        td {
          color: #525c6c;
          font-family: @main-font;
          font-weight: 400;
          font-size: 13px;
          line-height: 12px;
          &:nth-child(2) {
            text-align: left;
            border-right: none;
          }
          &:nth-child(3) {
            text-align: right;
            border-right: 1px solid #e9ecee;
          }
          &:last-child {
            text-align: left;
          }
        }
      }
      &:last-child {
        background-color: #fff;
        border-bottom: none;
      }
    }
    .btn-trade {
      position: relative;
      padding: 6px 8px 5px 22px;
      background-color: #1991eb;
      text-shadow: 1px 1px 0 rgba(0, 0, 0, 0.1);
      box-shadow: 0 1px 0 #1175c0;
      color: #fff;
      font-size: 11px;
      font-family: @main-font;
      font-weight: 500;
      line-height: 11px;
      &::before {
        content: "";
        position: absolute;
        width: 10px;
        height: 9px;
        left: 7px;
        top: 6px;
        background-image: @img-wallet;
        background-position: center;
        background-repeat: no-repeat;
        z-index: 11;
      }
    }
  }
</style>
