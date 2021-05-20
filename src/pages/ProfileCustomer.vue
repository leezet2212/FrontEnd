<template>
  <div>
    <div class="container">
      <div class="customer">
        <h1>Thông tin khách hàng</h1>
        <b-form @submit.prevent="SubmitAdd">
          <b-form-group
            id="input-group-2"
            label="Full Name:"
            label-for="input-2"
          >
            <b-form-input
              id="input-2"
              v-model="formadd.name"
              placeholder="Enter name"
              required
            >
            </b-form-input>
          </b-form-group>
          <b-form-group id="input-group-2" label="Phone:" label-for="input-2">
            <b-form-input
              id="input-2"
              v-model="formadd.phone"
              placeholder="Enter phone"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group id="input-group-2" label="Address:" label-for="input-2">
            <b-form-input
              id="input-2"
              v-model="formadd.address"
              placeholder="Enter address"
              required
            ></b-form-input>
          </b-form-group>
          <div class="update-btn">
            <b-button
              type="submit"
              @click="update()"
              class="btn-update"
              variant="primary"
              >Update</b-button
            >
          </div>
        </b-form>
      </div>
      <div class="history-product">
        <h2>Lịch sử mua hàng</h2>
        <table class="table category">
          <thead>
            <slot name="columns">
              <tr>
                <th>Mã Bill</th>
                <th>Tổng tiền</th>
                <th>Phương thức thanh toán</th>
                <th>Trạng thái</th>
              </tr>
            </slot>
          </thead>
          <tbody>
            <tr
              v-for="bill in bill1.data"
              v-bind:key="bill.id"
              v-if="bill.status == 1"
            >
              <td>{{ bill.id }}</td>
              <td>{{ bill.total }}</td>
              <td>{{ bill.payment }}</td>
              <td>
                <div v-if="bill.status == 1">
                  <b-badge variant="success">Chờ xác nhận</b-badge>
                </div>
                <div v-else-if="bill.status == 2">
                  <b-badge variant="danger">Đang vận chuyển</b-badge>
                </div>
                <div v-else>
                  <b-badge variant="warning">Gửi thành công</b-badge>
                </div>
              </td>
              <td>
                <b-button
                        variant="outline-info"
                        v-b-modal.modal-center
                        @click="show(bill.id)"
                        >Chi tiết</b-button
                      >
              </td>
            </tr>
          </tbody>
          <b-modal
                  size="xl"
                  id="modal-center"
                  ref="modal"
                  title="THÔNG TIN CHI TIẾT HÓA ĐƠN"
                  @show="resetModal"
                  @hidden="resetModal"
                  @ok="handleOk"
                >
                  <table class="table">
                    <thead>
                      <tr v-bind:key="id">
                        <th>
                          <table class="table">
                            <tr>
                              <th>Mã Bill</th>
                              <th>Sản phẩm</th>
                              <th>Hình ảnh</th>
                              <th>Màu</th>
                              <th>Size</th>
                              <th>Số lượng</th>
                              <th>Đơn giá</th>
                              <th>Tổng giá</th>
                            </tr>
                            <tr
                              v-for="bill0 in billDetail"
                              v-bind:key="bill0.id"
                            >
                              <td>{{ bill0.id_bill }}</td>
                              <td>{{ bill0.name }}</td>
                              <td>
                                <img
                                  v-bind:src="
                                    'http://127.0.0.1:8000/uploads/product/' +
                                      bill0.img
                                  "
                                  width="100"
                                  alt="product"
                                />
                              </td>
                              <td>{{ bill0.color }}</td>
                              <td>{{ bill0.size }}</td>
                              <td>{{ bill0.amount }}</td>
                              <td>{{ bill0.price }}</td>
                              <td>{{ bill0.price * bill0.amount }}</td>
                            </tr>
                          </table>
                        </th>
                      </tr>
                    </thead>
                  </table>
                </b-modal>
        </table>
      </div>
    </div>
    <top-nav-home />
    <content-footer-home />
  </div>
</template>

<script>
import ContentFooterHome from "../components/Footer/ContentFooterHome.vue";
import TopNavHome from "../layout/TopNavHome.vue";
import Vue from "vue";
import axios from "axios";
import VueAxios from "vue-axios";
import Swal from "sweetalert2";
Vue.use(VueAxios, axios);
export default {
  components: {
    ContentFooterHome,
    TopNavHome,
  },
  data() {
    return {
      bill1: [],
      formedit: {
        id: "",
        customer_id: "",
        total: "",
        payment: "",
        dateorder: null,
        note: 1,
        status: null,
      },
      formadd: {
        name: "",
        phone: "",
        address: "",
      },
      customer: [],
      billDetail: [],
    };
  },
  created() {
    this.getItem();
    this.getBill();
  },
  methods: {
    SubmitAdd() {
      var self = this;
      this.customer = JSON.parse(localStorage.getItem("customer"));
      var edit = this.formadd;
      axios
        .put(`http://127.0.0.1:8000/api/customer/` + this.customer.id, edit)
        .then((res) => {
          self.getItem();
          console.log("Thành công");
          Swal.fire("Đã update!", "Update thông tin thành công.", "success");
        })
        .catch((error) => {
          this.getItem();
          Swal.fire("Failed!", error, "warning");
        });
    },
    getItem() {
      var self = this;
      this.customer = JSON.parse(localStorage.getItem("customer"));
      Vue.axios
        .get("http://127.0.0.1:8000/api/customer/" + this.customer.id)
        .then(function (resp) {
          self.formadd.name = resp.data.data.name;
          self.formadd.phone = resp.data.data.phone;
          self.formadd.address = resp.data.data.address;
        })
        .catch(function (error) {
          console.log("Loi:", error);
        });
    },
    // bill
    show(id) {
      var self = this;
      this.customer = JSON.parse(localStorage.getItem("customer"));
      this.formedit.id = id;
      axios.get("http://127.0.0.1:8000/api/bill/" + id).then((res) => {
        self.name = res.data.customer.name;
        self.email = res.data.customer.email;
        self.phone = res.data.customer.phone;
        self.address = res.data.customer.address;
        self.billDetail = res.data.billdetail;
      });
    },
    getBill() {
      var self = this;
      this.customer = JSON.parse(localStorage.getItem("customer"));
      Vue.axios
        .get("http://127.0.0.1:8000/api/bill")
        .then(function (resp) {
          self.bill1 = resp.data;
          console.log("Data:", resp.data.data);
        })
        .catch(function (error) {
          console.log("Loi:", error);
        });
    },
  },
};
</script>

<style scoped>
.container {
  margin-top: 113px;
}
#input-group-2 {
  margin-bottom: 1rem;
  width: 40%;
  margin: 0 auto;
}
.container h1 {
  text-align: center;
}
.history-product {
  border-top: 1px solid black;
}
.history-product h2 {
  text-align: center;
}
.btn-update {
  margin: 27px auto;
  display: block;
}
</style>