<template>
  <main>
    <section class="calc noPrint">
      <div class="calc__content">
        <div class="container">
          <div class="calc__wrapper">
            <div class="calc__inner-data">
              <h1 class="calc__title">Формирование КП</h1>
              <Select2
                class="select-component"
                v-model="selectedManager"
                :options="managerOptions"
                :settings="{
                  placeholder: 'Ответственный менеджер',
                  minimumResultsForSearch: -1,
                }"
                @select="setManager(selectedManager)"
              />
              <Select2
                class="select-component"
                v-model="selectedTemplate"
                :options="templateOptions"
                :settings="{
                  placeholder: 'Шаблон КП',
                  minimumResultsForSearch: -1,
                }"
                @select="setOfferTmp($event)"
              />
              <label class="styles-input">
                <input
                  type="text"
                  autocomplete="off"
                  data-styles-field
                  placeholder=" "
                  v-model="offerTitle"
                />
                <span class="styles-text">Заголовок КП</span>
              </label>

              <div
                class="area-field"
                style="margin-top: 0; margin-bottom: 20px"
              >
                <textarea placeholder=" " v-model="offerDescr"></textarea>
                <span class="styles-text">Описание КП</span>
              </div>
              <div
                class="area-field"
                style="margin-top: 0; margin-bottom: 20px"
              >
                <textarea placeholder=" " v-model="footnote"></textarea>
                <span class="styles-text">Сноска под таблицей</span>
              </div>

              <label class="styles-input">
                <input
                  type="text"
                  autocomplete="off"
                  data-styles-field
                  placeholder=" "
                  v-model="clientName"
                />
                <span class="styles-text">Наименование клиента</span>
              </label>

              <label class="styles-input">
                <input
                  type="number"
                  autocomplete="off"
                  data-styles-field
                  placeholder=" "
                  v-model="projectDeadline"
                />
                <span class="styles-text">Срок выполнения проекта</span>
              </label>

              <label class="styles-input">
                <input
                  type="number"
                  autocomplete="off"
                  data-styles-field
                  placeholder=" "
                  v-model="discountPrecent"
                  @input="calcResult"
                />
                <span class="styles-text">Скидка на разработку %</span>
              </label>

              <div class="solution">
                <div class="wrap">
                  <span
                    class="solution-title"
                    style="line-height: 120%; margin-bottom: 20px"
                    >Добавить услугу</span
                  >
                </div>
                <label class="styles-input styles-input__title">
                  <input
                    type="text"
                    data-styles-field
                    placeholder=" "
                    id="solutionTitle"
                    v-model="serviceTitle"
                  />
                  <span class="styles-text">Заголовок услуги</span>
                </label>
                <div class="area-field">
                  <textarea
                    placeholder=" "
                    id="solutionArea"
                    v-model="serviceDescription"
                  ></textarea>
                  <span class="styles-text">Описание услуги</span>
                </div>
                <div class="solution__footer">
                  <label class="styles-input">
                    <input
                      type="number"
                      data-styles-field
                      placeholder=" "
                      v-model="servicePrice"
                    />
                    <span class="styles-text">Стоимость услуги</span>
                  </label>
                  <label class="radio-switch">
                    <span>Фикс</span>
                    <input
                      checked
                      type="radio"
                      name="period"
                      value="fix"
                      @input="getPeriod"
                    />
                  </label>
                  <label class="radio-switch">
                    <span>В месяц</span>
                    <input
                      type="radio"
                      name="period"
                      value="month"
                      @input="getPeriod"
                    />
                  </label>
                  <label class="radio-switch">
                    <span>В год</span>
                    <input
                      type="radio"
                      name="period"
                      value="year"
                      @input="getPeriod"
                    />
                  </label>
                  <button class="btn" @click="addService">
                    Добавить в смету
                  </button>
                </div>
              </div>
            </div>
            <transition name="fade" mode="out-in">
              <div class="calc__outer-data" v-show="services.length">
                <h2 class="outer-data__title">Предпросмотр КП</h2>
                <div class="result-list__shadow">
                  <div class="result-list__wrap">
                    <draggable
                      class="result-list"
                      :list="services"
                      v-bind="dragOptions"
                      item-key="serviceId"
                      handle=".handle"
                    >
                      <template #item="{ element }">
                        <div
                          class="list-item"
                          :class="{ editing: element.isEdited }"
                        >
                          <div class="handle">
                            <span></span>
                            <span></span>
                            <span></span>
                          </div>
                          <div class="list-item__wrapper">
                            <div class="list-item__head">
                              <div class="list-item__block list-item__name">
                                <span class="list-item__nameplate"
                                  >Наименование услуги</span
                                >
                                <div
                                  class="list-item__data"
                                  v-show="!element.isEdited"
                                >
                                  {{ element.serviceTitle }}
                                </div>
                                <input
                                  type="text"
                                  v-show="element.isEdited"
                                  v-model="element.serviceTitle"
                                  class="editing-input"
                                />
                              </div>
                              <div class="list-item__block">
                                <span class="list-item__nameplate"
                                  >Стоимость</span
                                >
                                <div
                                  class="list-item__data is-price"
                                  v-show="!element.isEdited"
                                >
                                  {{
                                    element.servicePrice.toLocaleString("RU-ru")
                                  }}
                                  ₸
                                </div>
                                <input
                                  type="number"
                                  v-show="element.isEdited"
                                  v-model="element.servicePrice"
                                  class="editing-input"
                                />
                              </div>
                              <div
                                class="list-item__delete"
                                @click="deleteservice(element.serviceId)"
                              ></div>
                              <div
                                class="list-item__edit"
                                @click="
                                  (element.isEdited = !element.isEdited),
                                    saveСhanges(element)
                                "
                              ></div>
                            </div>
                            <div class="list-item__footer">
                              <div
                                class="list-item__descr"
                                v-html="element.serviceDescription"
                                v-show="!element.isEdited"
                              ></div>
                              <textarea
                                class="editing-area"
                                v-model="element.modifyServiceDescription"
                                v-show="element.isEdited"
                              ></textarea>
                            </div>
                          </div>
                        </div>
                      </template>
                    </draggable>
                  </div>
                </div>
                <div class="calc-result">
                  <div class="field-price" v-if="discountPrecent">
                    Стоимость:<span>{{
                      offerPrice.toLocaleString("RU-ru")
                    }}</span
                    >₸ {{ periodText }}
                  </div>
                  <div class="field-sale" v-if="discountPrecent">
                    Скидка {{ discountPrecent }} % :<span>{{
                      ((offerPrice / 100) * discountPrecent).toLocaleString(
                        "RU-ru"
                      )
                    }}</span
                    >₸
                  </div>
                  <div class="field-total-price">
                    Итоговая стоимость:<span>{{
                      offerPriceWithDiscount.toLocaleString("RU-ru")
                    }}</span
                    >₸ {{ periodText }}
                  </div>
                </div>
                <div class="btns-wrapper">
                  <button class="btn" @click="getPrint()">
                    Сформировать КП
                  </button>
                  <button class="btn" @click="getPrintWidthoutPrice()">
                    КП без цен
                  </button>
                  <button class="btn" @click="getPrintWidthoutSeal()">
                    КП без печати
                  </button>
                </div>
              </div>
            </transition>
          </div>
        </div>
      </div>
    </section>
    <section id="offer-doc" class="offer-doc" v-show="toPrint">
      <div class="container">
        <div class="offer-doc__head">
          <div class="offer-doc__logo">
            <img src="../src/assets/img/logo.svg" alt="" />
          </div>
          <div class="offer-doc__company-data">
            <span>{{ selectedManagerObject.fullName }}</span>
            <span>{{ selectedManagerObject.phone }}</span>
            <a
              :href="`mailto:${selectedManagerObject.email}`"
              class="blue-tdu"
              >{{ selectedManagerObject.email }}</a
            >
          </div>
        </div>
        <div class="offer-doc__title">{{ offerTitle }}</div>
        <div class="offer-doc__info-fields">
          <div class="offer-doc__info-field">
            Дата составления КП:
            <span class="bold-text">{{ getCurrentDate() }}</span>
          </div>
          <div class="offer-doc__info-field">
            Клиент:
            <span class="bold-text">{{ clientName }}</span>
          </div>
        </div>
        <div style="white-space: pre-line" class="offer-doc__descr">
          {{ offerDescr }}
        </div>
        <span class="offer-title">Что будет реализовано:</span>
        <table class="offer-doc__table">
          <thead>
            <tr :class="[period, printWidthoutPrice]" class="thead-row">
              <th>Услуга</th>
              <th>Описание</th>
              <th>Стоимость</th>
            </tr>
          </thead>
          <tbody>
            <tr
              :class="[period, printWidthoutPrice]"
              class="tbody-row"
              v-for="service in services"
              :key="service.serviceId"
            >
              <td>
                <span class="setvise-table-title">{{
                  service.serviceTitle
                }}</span>
              </td>
              <td v-html="service.serviceDescription"></td>
              <td class="center-text">
                {{ service.servicePrice.toLocaleString("RU-ru") }} ₸
              </td>
            </tr>
          </tbody>
        </table>
        <div
          style="white-space: pre-line"
          v-if="footnote.length"
          class="offer-doc__descr cursive"
        >
          * {{ footnote }}
        </div>
        <div class="offer-doc__footer">
          <span class="boss-name">
            С уважением,<br />{{ selectedManagerObject.text }}
          </span>
          <div class="offer-doc__seal" v-if="!printWidthoutSeal">
            <img src="../src/assets/img/seal.png" alt="" />
          </div>
          <div class="offer-doc__res">
            <div class="res-block" v-if="discountPrecent">
              Итого:&nbsp;<span>{{ offerPrice.toLocaleString("RU-ru") }}</span
              >&nbsp;<span>₸ {{ periodText }}</span>
            </div>
            <div class="res-block" v-if="discountPrecent">
              Скидка {{ discountPrecent }} % :&nbsp;<span>{{
                ((offerPrice / 100) * discountPrecent).toLocaleString("RU-ru")
              }}</span
              >&nbsp;₸
            </div>
            <div class="res-block" v-if="projectDeadline">
              Срок разработки:&nbsp;
              <span>{{ projectDeadline }} рабочих дней</span>
            </div>
            <div class="res-block">
              Итоговая стоимость:&nbsp;<span>{{
                offerPriceWithDiscount.toLocaleString("RU-ru")
              }}</span
              >&nbsp;<span>₸ {{ periodText }}</span>
            </div>
          </div>
        </div>
      </div>
    </section>
  </main>

  <div class="login-modal show" v-if="!isAuthorized">
    <div class="login-modal__content">
      <div class="login-modal__form">
        <label class="modal-field">
          <span class="modal-field__text"></span>
          <input
            type="password"
            name="password"
            v-model="authPass"
            v-on:keyup.enter="checkPass"
            placeholder="Введите пароль"
          />
        </label>
        <button type="button" @click="checkPass" class="btn">
          Авторизация
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import draggable from "vuedraggable";
import Select2 from "vue3-select2-component";
import Localdb from "../src/assets/localDb.json";

export default {
  name: "App",
  components: {
    draggable,
    Select2,
  },
  data() {
    return {
      isAuthorized: false,
      authPassData: "Voxel987=",
      authPass: "",
      services: [],
      offerTitle: "",
      offerDescr: "",
      offerPrice: 0,
      discountPrecent: null,
      offerPriceWithDiscount: 0,
      clientName: "",
      projectDeadline: "",
      serviceTitle: "",
      serviceDescription: "",
      servicePrice: "",
      period: "fix",
      periodText: "",
      toPrint: false,
      footnote: "",
      selectedTemplate: null,
      templateOptions: Localdb.templates,
      managerOptions: Localdb.managers,
      selectedManager: null,
      selectedManagerObject: Localdb.managers[0],
      printWidthoutPrice: "",
      printWidthoutSeal: false,
    };
  },

  watch: {},
  created() {
    if (sessionStorage.getItem("loginTrue")) {
      this.isAuthorized = true;
    }
  },
  computed: {
    dragOptions() {
      return {
        scrollSensitivity: 200,
        forceFallback: true,
        animation: 400,
        group: "pollTypes",
        disabled: false,
        ghostClass: "ghost",
        sort: true,
      };
    },
  },
  methods: {
    setManager(id) {
      this.selectedManagerObject = this.managerOptions.find(
        (item) => item.id == id
      );
      console.log(this.selectedManagerObject);
    },
    checkPass() {
      this.isAuthorized =
        this.authPass.trim() === this.authPassData ? true : false;

      if (this.isAuthorized) {
        sessionStorage.setItem("loginTrue", true);
      }
    },

    setOfferTmp(event) {
      const serviceList = event.services;
      this.services = [];
      if (serviceList.length) {
        serviceList.forEach((service) => {
          this.services.push({
            serviceTitle: service.serviceTitle,
            serviceDescription: service.serviceDescription,
            modifyServiceDescription: service.serviceDescription,
            servicePrice: service.servicePrice,
            serviceId: Math.random().toString(36).substring(2, 9),
            isEdited: false,
          });
        });
        this.calcResult();
      }
    },

    setPeriodText() {
      switch (this.period) {
        case "fix":
          this.periodText = "";
          break;
        case "year":
          this.periodText = "/ год";
          break;
        case "month":
          this.periodText = "/ мес";
          break;
        default:
          break;
      }
    },

    getPeriod(event) {
      let period = event.target.value;
      this.period = period;
      this.setPeriodText();
    },

    saveСhanges(service) {
      this.calcResult();
      this.services.forEach((item) => {
        if (item.serviceId === service.serviceId) {
          item.serviceDescription = service.modifyServiceDescription
            .split("\n")
            .filter((words) => words.trim().length > 0)
            .map((words) => {
              if (words.length > 0) {
                return `${words}<br>`;
              }
            })
            .join("");
        }
      });
    },

    addService() {
      if (
        !this.serviceTitle.trim() ||
        !this.serviceDescription.trim() ||
        !this.servicePrice === ""
      ) {
        return;
      }
      let modifyServiceDescription = this.serviceDescription;
      let serviceDescription = this.serviceDescription
        .split("\n")
        .filter((words) => words.trim().length > 0)
        .map((words) => {
          if (words.length > 0) {
            return `${words}<br>`;
          }
        })
        .join("");

      this.services.push({
        serviceTitle: this.serviceTitle,
        serviceDescription,
        modifyServiceDescription,
        servicePrice: this.servicePrice,
        serviceId: Math.random().toString(36).substring(2, 9),
        isEdited: false,
      });

      this.serviceTitle = "";
      this.serviceDescription = "";
      this.servicePrice = "";
      this.serviceId = "";

      this.calcResult();
    },

    calcResult() {
      this.offerPrice = this.services.reduce((acc, service) => {
        return (acc += +service.servicePrice);
      }, 0);
      this.offerPriceWithDiscount =
        this.offerPrice - (this.offerPrice / 100) * this.discountPrecent;
    },

    deleteservice(choiseServiceId) {
      this.services = this.services.filter(
        (service) => service.serviceId !== choiseServiceId
      );
      this.calcResult();
    },

    getPrint() {
      this.toPrint = !this.toPrint;
      setTimeout(() => {
        window.print();
        this.toPrint = !this.toPrint;
        if (this.printWidthoutPrice !== "") this.printWidthoutPrice = "";
        if (this.printWidthoutSeal === true) this.printWidthoutSeal = false;
      }, 200);
    },

    getPrintWidthoutPrice() {
      this.printWidthoutPrice = "hide";
      this.getPrint();
    },

    getPrintWidthoutSeal() {
      this.printWidthoutSeal = true;
      this.getPrint();
    },

    getCurrentDate() {
      let Data = new Date();
      let Year = Data.getFullYear();
      let Month = Data.getMonth();
      let Day = Data.getDate();
      let fMonth;
      switch (Month) {
        case 0:
          fMonth = "января";
          break;
        case 1:
          fMonth = "февраля";
          break;
        case 2:
          fMonth = "марта";
          break;
        case 3:
          fMonth = "апреля";
          break;
        case 4:
          fMonth = "мае";
          break;
        case 5:
          fMonth = "июня";
          break;
        case 6:
          fMonth = "июля";
          break;
        case 7:
          fMonth = "августа";
          break;
        case 8:
          fMonth = "сентября";
          break;
        case 9:
          fMonth = "октября";
          break;
        case 10:
          fMonth = "ноября";
          break;
        case 11:
          fMonth = "декабря";
          break;
      }
      return Day + " " + fMonth + " " + Year + " года";
    },
  },
};
</script>
<style>
.list-item.sortable-chosen.ghost {
  opacity: 0;
}

.list-item.sortable-chosen.sortable-fallback.sortable-drag {
  background: #17171a;
  cursor: grab;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease-in-out;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
