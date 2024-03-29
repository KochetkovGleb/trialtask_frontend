<template>
  <div>
    <p v-if="tariffExists">
      Тариф на {{ displayDate }} - {{ monthPrice }} руб. за кубометр воды.
    </p>
    <el-form ref="form">
      <el-form-item label="Выберите месяц и год:">
        <el-date-picker
          class="datePicker"
          ref="elDatePicker"
          v-model="tariff.begin_date"
          format="MMMM-yyyy"
          :value-format="valueFormat"
          type="month"
          placeholder="Дата"
          :picker-options="pickerOptions"
          @change="
            checkTariffExists();
            clearError('begin_date');
          "
        ></el-date-picker>
      </el-form-item>
      <p class="errorMessage">{{ errors.begin_date }}</p>
      <el-form-item label="Цена за кубометр воды в рублях:">
        <el-input
          placeholder="0.00"
          v-model="tariff.price"
          clearable
          type="number"
          min="0"
          step="0.01"
          @input="clearError('price')"
        >
        </el-input>
      </el-form-item>
      <p class="errorMessage mb-20">{{ errors.price }}</p>
      <el-form-item>
        <el-button type="primary" @click.prevent="save">Сохранить</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { dateMixin } from "../mixins/dateMixin.js";
import { errorMixin } from "../mixins/errorMixin";

export default {
  name: "Tariffs",
  mixins: [dateMixin, errorMixin],
  data() {
    return {
      tariffExists: false,
      monthPrice: null,
      errors: {
        begin_date: null,
        price: null,
      },
      tariff: {
        id: null,
        price: null,
        begin_date: null,
      },
    };
  },
  methods: {
    checkTariffExists() {
      this.tariffExists = false;
      this.monthPrice = null;

      this.tariff.price = null;

      if (!this.tariff.begin_date) {
        return;
      }

      this.axios
        .get(`/api/tariffs?begin_date=${this.tariff.begin_date}`)
        .then((response) => {
          if (response.data.data) {
            this.displayDate = this.$refs.elDatePicker.displayValue.replace(
              /-/,
              " "
            );

            this.tariffExists = true;
            this.tariff = response.data.data;
            this.errors.price = null;
            this.monthPrice = this.tariff.price;
          }
        })
        .catch((error) => console.log(error));
    },
    save() {
      let method = this.tariffExists ? "patch" : "post";
      let url = "api/tariffs";

      if (method === "patch") {
        url += "/" + this.tariff.id;
      }

      this.axios[method](url, {
        begin_date: this.tariff.begin_date,
        price: this.tariff.price,
      })
        .then((response) => {
          if (method === "post") {
            this.tariffExists = true;
          }

          this.tariff = response.data.data;
          this.monthPrice = this.tariff.price;

          this.displayDate = this.$refs.elDatePicker.displayValue.replace(
            /-/,
            " "
          );

          this.$message({
            showClose: true,
            message: "Тариф успешно установлен!",
            type: "success",
          });
        })
        .catch((error) =>
          this.setErrors(error.response.data.errors, "price", "begin_date")
        );
    },
  },
};
</script>

<style>
.el-form-item__label {
  float: none;
}

.el-input {
  width: 220px;
}
</style>
