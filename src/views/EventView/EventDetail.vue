<template>
  <div class="event-container" v-if="event">
    <div class="event-header">
      <h1 class="event-title">{{ event.eventTitle }}</h1>
      <p v-for="(tag, index) in eventTags" :key="index" class="event-tags">
        {{ tag }}
      </p>
      <p class="event-age-tag">{{ event.eventAgeTag }}</p>
      <div class="stars">
        <span
          v-for="index in getStars(event).fullStars"
          :key="'full-' + index"
          class="star full-star"
        >
          ★
        </span>
        <span v-if="getStars(event).halfStars" class="star half-star">★</span>
        <span
          v-for="index in getStars(event).emptyStars"
          :key="'empty-' + index"
          class="star empty-star"
        >
          ★
        </span>
      </div>
    </div>

    <div class="event-content">
      <swiper
        v-if="event?.eventListImgURL?.length"
        :space-between="10"
        :slides-per-view="1"
        :loop="true"
        :autoplay="{ delay: 3000 }"
        class="event-image-carousel"
        navigation
      >
        <swiper-slide
          v-for="(image, index) in event.eventListImgURL"
          :key="index"
          class="event-image-slide"
        >
          <img :src="image" alt="Event Image" />
        </swiper-slide>
      </swiper>

      <div class="event-details card shadow-lg border-0 rounded-4">
        <div class="card-body p-4">
          <h3
            class="card-title text-center text-uppercase fw-bold text-primary mb-4"
          >
            <i class="fas fa-calendar-alt"></i> Chi tiết sự kiện
          </h3>
          <ul class="list-group list-group-flush">
            <li class="list-group-item d-flex justify-content-between">
              <strong
                ><i class="fas fa-clock text-warning"></i> Ngày bắt đầu:</strong
              >
              <span>{{ formattedStartDate }}</span>
            </li>
            <li class="list-group-item d-flex justify-content-between">
              <strong
                ><i class="fas fa-clock text-danger"></i> Ngày kết thúc:</strong
              >
              <span>{{ formattedEndDate }}</span>
            </li>
            <li class="list-group-item d-flex justify-content-between">
              <strong
                ><i class="fas fa-money-bill-wave text-success"></i> Giá
                gốc:</strong
              >
              <span class="fw-semibold text-success">{{
                formatCurrency(event.eventPrice)
              }}</span>
            </li>
            <li class="list-group-item d-flex justify-content-between">
              <strong
                ><i class="fas fa-map-marker-alt text-info"></i> Địa chỉ tổ
                chức:</strong
              >
              <span>{{ event.eventAddress }}</span>
            </li>
            <li class="list-group-item d-flex justify-content-between">
              <strong
                ><i class="fas fa-users text-primary"></i> Sức chứa (tối
                đa):</strong
              >
              <span class="fw-semibold text-primary"
                >{{ event.eventCapacity }} người</span
              >
            </li>
            <li class="list-group-item d-flex justify-content-between">
              <strong
                ><i class="fas fa-check-circle text-secondary"></i> Trạng
                thái:</strong
              >
              <span class="fw-bold">{{ event.eventStatus }}</span>
            </li>
          </ul>

          <button
            class="btn btn-primary mt-2 mx-2"
            :disabled="!canUpdateZone"
            @click="isModalOpen = true"
            :title="
              !canUpdateZone
                ? 'Chỉ được cập nhật zone trước 7 ngày trước khi sự kiện bắt đầu'
                : ''
            "
          >
            Cập nhật Giá vé
          </button>
          <router-link
            v-if="event.eventStatus === 'UP_COMMING'"
            :to="{
              name: 'EventUpdate',
              params: {
                eventId: event.eventId,
                companyId: event.eventCompanyId,
              },
            }"
            class="btn btn-danger position-relative mt-2 mx-2"
            :class="{ disabled: !canUpdateZone }"
            v-tooltip="
              !canUpdateZone
                ? 'Chỉ được cập nhật trước 7 ngày trước khi sự kiện bắt đầu'
                : ''
            "
          >
            <i class="fas fa-edit"></i> Gửi đơn xét duyệt
          </router-link>

          <button
            v-if="
              (event.eventStatus === 'AWAITING_APPROVAL' ||
                event.eventStatus === 'CANCELLED') &&
              canUpdateZone
            "
            class="btn btn-danger position-relative mt-2 mx-2"
            :class="{ disabled: !canUpdateZone }"
            v-tooltip="
              !canUpdateZone
                ? 'Chỉ được cập nhật trước 7 ngày trước khi sự kiện bắt đầu'
                : ''
            "
            @click="deleteEvent(event.eventId)"
          >
            <i class="fa-solid fa-trash"></i> Xóa Sự Kiện
          </button>

          <p class="text-sm text-red-500 mb-4 mt-2">
            *Chỉ có thể cập nhật sự kiện trước 7 ngày trước khi sự kiện bắt đầu
            <br />
            *Chỉ có thể xóa sự kiện trước 7 ngày trước khi sự kiện bắt đầu hoặc
            sự kiện đã kết thúc hoặc đã hủy (Mọi thông tin sự kiện sẽ bị xóa
            vĩnh viễn bao gồm vé)
          </p>
        </div>
      </div>
    </div>

    <div class="event-descriptions">
      <h2 class="text-3xl font-bold text-white mb-6 text-center mt-4">
        Mô Tả Sự Kiện
      </h2>
      <p class="event-description">
        {{ isExpanded ? event.eventDescription : truncatedText }}
      </p>
      <button
        v-if="event.eventDescription.length > maxLength"
        @click="isExpanded = !isExpanded"
        class="toggle-btn"
      >
        {{ isExpanded ? "Thu gọn" : "Xem thêm" }}
      </button>
    </div>

    <div class="event-tickets">
      <h2 class="text-3xl font-bold text-white mb-6 text-center">
        Thông Tin Vé
      </h2>
      <div class="ticket-list">
        <div
          v-for="(remainingCapacity, day) in event.eventTicketCapacity"
          :key="day"
          class="ticket-card"
        >
          <p>
            <strong>DAY {{ day }}</strong>
          </p>
          <p>Số lượng vé còn lại: {{ remainingCapacity }}</p>
          <!-- <button
            class="book-btn"
            :disabled="!canPurchaseTicket"
            @click="openModal({ day, remainingCapacity })"
            :title="
              !canPurchaseTicket
                ? 'Vé chỉ được mua trong vòng 7 ngày trước khi sự kiện bắt đầu'
                : ''
            "
          >
            Đặt vé ngay
          </button> -->
        </div>
        <div class="ticket-card">
          <p>
            <strong>FULL DAY</strong>
          </p>
          <p>Tổng: {{ event.totalDay }} ngày</p>
          <!-- <button
            class="book-btn"
            :disabled="!canPurchaseTicket"
            @click="openModalAllDay"
            :title="
              !canPurchaseTicket
                ? 'Vé chỉ được mua trong vòng 7 ngày trước khi sự kiện bắt đầu'
                : ''
            "
          >
            Đặt vé toàn sự kiện
          </button> -->
        </div>
      </div>
    </div>

    <UpdateZoneModal
      :eventId="event.eventId"
      :isModalOpen="isModalOpen"
      @close="isModalOpen = false"
    />
    <!-- Modal đặt vé -->
  </div>
  <!-- đánh giá -->
  <div class="container mt-7 border border-black p-4 rounded-lg">
    <h4
      class="mb-4 text-2xl font-extrabold text-gray-900 dark:text-white md:text-5xl lg:text-6xl"
    >
      Phần đánh giá
    </h4>

    <loading :active="loadingFeedbacks" />
    <div
      v-if="feedbacks.length === 0 && !loadingFeedbacks"
      class="text-center text-gray-500"
    >
      Chưa có đánh giá nào.
    </div>
    <div v-else class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <div
        v-for="feedback in feedbacks"
        :key="feedback.fbId"
        class="p-6 border rounded-lg shadow-md bg-gray-50"
      >
        <h3 class="text-xl font-semibold text-gray-800 mb-2">
          Đánh giá từ vé số: {{ feedback.ticketId }}
        </h3>
        <div class="flex items-center mb-2">
          <span
            class="text-yellow-500 mr-1"
            v-for="i in feedback.fbRate"
            :key="i"
            >⭐</span
          >
          <span
            class="text-gray-500"
            v-for="i in 5 - feedback.fbRate"
            :key="`empty-${i}`"
            >☆</span
          >
        </div>

        <p class="text-gray-700">
          {{ feedback.fbContent }}
        </p>
        <p class="text-gray-600 text-sm mt-2">
          Ngày đánh giá:
          <span class="font-semibold">
            {{ formatDate(feedback.fbCreateDate) }}
          </span>
        </p>
      </div>
    </div>

    <!-- Pagination -->
    <div
      v-if="totalFeedbackPages > 1"
      class="flex justify-between items-center mt-6"
    >
      <button
        @click="prevFeedbackPage"
        :disabled="feedbackPage === 1"
        class="px-4 py-2 bg-blue-500 text-gray-700 rounded-lg disabled:opacity-50"
      >
        Trang trước
      </button>
      <span class="text-gray-800 font-medium"
        >Trang {{ feedbackPage }} / {{ totalFeedbackPages }}</span
      >
      <button
        @click="nextFeedbackPage"
        :disabled="feedbackPage === totalFeedbackPages"
        class="px-4 py-2 bg-blue-500 text-gray-700 rounded-lg"
      >
        Trang sau
      </button>
    </div>
  </div>
  <!-- bài đăng -->
  <div class="container mt-7 border border-black p-4 rounded-lg">
    <h4
      class="mb-4 text-2xl font-extrabold text-gray-900 dark:text-white md:text-5xl lg:text-6xl"
    >
      Phần bài đăng
    </h4>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
      <div
        v-for="blog in blogs"
        :key="blog.blogId"
        class="border border-gray-300 p-4 rounded-lg shadow-sm"
      >
        <!-- Nội dung blog -->
        <div class="flex justify-between items-center">
          <h2 class="text-lg font-semibold">
            {{ blog.blogUserId || "Ẩn danh" }}
          </h2>
          <p class="text-sm text-gray-500">
            {{ new Date(blog.blogCreateDate).toLocaleDateString() }}
          </p>
        </div>
        <p class="text-gray-700 mt-1 truncate">{{ blog.blogContent }}</p>

        <!-- Hình ảnh nếu có -->
        <div
          v-if="blog.eventListImgURL && blog.eventListImgURL.length"
          class="mt-2"
        >
          <img
            :src="blog.eventListImgURL[0]"
            class="w-full h-52 object-cover rounded-lg"
            alt="Blog Image"
          />
        </div>

        <!-- Cảm xúc -->
        <div class="flex items-center gap-4 mt-2 text-gray-500">
          <button
            :disabled="this.$authStore.role == 'ADMIN'"
            @click="likeBlog(blog.blogId)"
            class="flex items-center gap-2 text-gray-500 hover:text-red-500 transition"
          >
            ❤️ {{ blog.blogEmotionsNumber }}
          </button>
          <button
            @click="goBlogDetail(blog.blogId)"
            class="flex items-center gap-1 hover:text-blue-500"
          >
            💬 <span>Xem Bình Luận</span>
          </button>
        </div>
      </div>
    </div>

    <div class="flex justify-between items-center mt-4">
      <button
        @click="fetchBlogs(currentPage - 1)"
        :disabled="currentPage === 0"
        class="px-4 py-2 bg-gray-300 rounded disabled:opacity-50"
      >
        ⬅ Trước
      </button>

      <span class="text-gray-700"
        >Trang {{ currentPage + 1 }} / {{ totalPages }}</span
      >

      <button
        @click="fetchBlogs(currentPage + 1)"
        :disabled="currentPage >= totalPages - 1"
        class="px-4 py-2 bg-gray-300 rounded disabled:opacity-50"
      >
        Sau ➡
      </button>
    </div>
  </div>
</template>

<script>
import { Swiper, SwiperSlide } from "swiper/vue";
import "swiper/swiper-bundle.css";
import { api } from "@/api/Api";
import UpdateZoneModal from "@/views/EventView/UpdateZoneModal.vue";

import { formatCurrency } from "@/composable/format";
import Loading from "vue-loading-overlay";
import "vue-loading-overlay/dist/css/index.css";
import Swal from "sweetalert2";
export default {
  components: {
    Swiper,
    SwiperSlide,
    UpdateZoneModal,
    Loading,
  },
  data() {
    return {
      blogs: [],
      currentPage: 0,
      totalPages: 1,

      feedbacks: [],
      loadingFeedbacks: false,
      feedbackPage: 1,
      feedbackSize: 10,
      totalFeedbackPages: 1,

      isModalOpen: false,
      loading: true,
      eventId: this.$route.params.eventId,
      event: null,
      showModal: false,
      isExpanded: false,
      maxLength: 100, // Giới hạn số ký tự hiển thị ban đầu
      showModalAllDay: false,
      string: "ALL_DAYS",
      selectedTicket: null,
    };
  },
  async mounted() {
    await this.fetchEventData();
    await this.fetchBlogs();
    await this.fetchFeedbacks();
  },
  methods: {
    goBlogDetail(blogId) {
      this.$router.push(
        `/company/${this.event.eventCompanyId}/events/${this.event.eventId}/blogs/${blogId}`
      );
    },
    async deleteEvent(eventId) {
      const result = await Swal.fire({
        title: "Xác nhận xóa?",
        text: "Bạn có chắc chắn muốn xóa hoàn toàn sự kiện này không?",
        icon: "warning",
        showCancelButton: true,
        confirmButtonText: "Xóa",
        cancelButtonText: "Hủy",
      });
      const companyId = this.event.eventCompanyId;
      if (result.isConfirmed) {
        api
          .delete(`/events/${eventId}`)
          .then(() => {
            this.$toast.success("Xóa sự kiện thành công");
            setTimeout(() => {
              this.$router.push(`/company/${companyId}/events`);
            }, 1000);
          })
          .catch((error) => {
            this.$toast.error(
              error.response?.data?.message || "Lỗi khi xóa sự kiện"
            );
          });
      }
    },
    async likeBlog(blogId) {
      try {
        const userId = this.event.eventCompanyId; // Lấy userId từ dữ liệu hiện có
        const res = await api.post(`/blogs/${blogId}/emotion`, null, {
          params: { userId }, // Gửi userId qua query parameters
        });
        const updatedBlog = res.data;
        console.log(updatedBlog);

        // Cập nhật lại danh sách blogs
        this.blogs = this.blogs.map((blog) =>
          blog.blogId === blogId ? { ...blog, ...updatedBlog } : blog
        );
      } catch (error) {
        this.$toast.error(
          error.response?.data?.message || "Lỗi khi thích bài viết"
        );
      }
    },
    async fetchBlogs(page = 0) {
      this.loading = true;
      this.error = null;
      try {
        const params = {};

        params.userId = this.event.eventCompanyId;
        params.eventId = this.event.eventId;
        params.month = null;
        params.year = null;

        console.log(params);
        params.page = page;
        params.size = 3;

        console.log("Params gửi API:", params);

        const response = await api.get("/blogs/filter", { params });
        this.blogs = response.data.data.content;
        this.currentPage = response.data.data.number;
        this.totalPages = response.data.data.totalPages;
      } catch (err) {
        this.$toast.error(err.response?.data?.message || "Lỗi khi gửi blog");
      } finally {
        this.loading = false;
      }
    },
    async fetchFeedbacks(page = 1) {
      this.loadingFeedbacks = true;
      this.error = null;
      try {
        const response = await api.get(
          `/blogs/feedbacks/event/${this.eventId}?page=${page - 1}&size=${
            this.feedbackSize
          }`
        );
        this.feedbacks = response.data.data.content;
        this.totalFeedbackPages = response.data.data.totalPages;
      } catch (error) {
        this.error =
          error.response?.data?.message || "Failed to fetch feedbacks";
        this.$toast.error(this.error);
      } finally {
        this.loadingFeedbacks = false;
      }
    },
    formatDate(dateString) {
      if (!dateString) return "Chưa xác định";
      return new Date(dateString).toLocaleDateString();
    },
    prevFeedbackPage() {
      if (this.feedbackPage > 1) {
        this.feedbackPage--;
        this.fetchFeedbacks(this.feedbackPage);
      }
    },
    nextFeedbackPage() {
      if (this.feedbackPage < this.totalFeedbackPages) {
        this.feedbackPage++;
        this.fetchFeedbacks(this.feedbackPage);
      }
    },
    formatCurrency,
    async fetchEventData() {
      this.loading = true;
      try {
        const response = await api.get(`/events/${this.eventId}`);
        this.event = response.data.data;
        console.log(this.event);
      } catch (error) {
        console.error("Error fetching event data:", error);
        this.$toast.error(error.response?.data?.message || "Đã xảy ra lỗi");
      } finally {
        this.loading = false;
      }
    },
    closeModal() {
      this.showModal = false;
      this.showModalAllDay = false;
      this.selectedTicket = null;

      // Quay lại trang event chính
      this.$router.push({
        path: `/company/events/${this.event.eventId}`,
      });
    },
    calculateAverageRating(event) {
      let totalReviews = 0;
      let weightedSum = 0;

      // Loop through ratings to calculate the weighted sum
      for (let star in event.eventRatingStart) {
        let count = event.eventRatingStart[star];
        weightedSum += star * count;
        totalReviews += count;
      }

      // Return the average rating or 0 if no ratings
      return totalReviews === 0 ? 0 : weightedSum / totalReviews;
    },

    // Get full, half, and empty stars based on the average rating of an event
    getStars(event) {
      const averageRating = this.calculateAverageRating(event);
      const fullStars = Math.floor(averageRating); // Full stars
      const halfStars = averageRating % 1 >= 0.5 ? 1 : 0; // Half star if needed
      const emptyStars = 5 - fullStars - halfStars; // Empty stars

      return {
        fullStars,
        halfStars,
        emptyStars,
      };
    },
  },
  computed: {
    truncatedText() {
      return this.event.eventDescription.length > this.maxLength
        ? this.event.eventDescription.substring(0, this.maxLength) + "..."
        : this.event.eventDescription;
    },
    formattedStartDate() {
      return this.event?.eventStartDate
        ? new Date(this.event.eventStartDate).toLocaleString()
        : "";
    },
    formattedEndDate() {
      return this.event?.eventEndDate
        ? new Date(this.event.eventEndDate).toLocaleString()
        : "";
    },
    eventTags() {
      return this.event?.eventTags?.split("|") || [];
    },
    totalRemainCapacity() {
      if (!this.event?.eventTicketCapacity) return 0;
      return Math.min(
        ...Object.values(this.event.eventTicketCapacity).map(
          (capacity) => capacity || Infinity
        )
      );
    },
    // Cho phép cập nhật Zone nếu sự kiện cách hiện tại hơn 7 ngày
    canUpdateZone() {
      if (
        !this.event ||
        !this.event.eventStartDate ||
        this.event.eventStatus === "CANCELLED"
      )
        return false;

      const now = new Date();
      const eventDate = new Date(this.event.eventStartDate);
      const diffTime = eventDate.getTime() - now.getTime();
      const diffDays = diffTime / (1000 * 3600 * 24);
      return diffDays > 7;
    },
    // Cho phép mua vé nếu thời gian còn lại từ 0 đến 7 ngày trước khi sự kiện bắt đầu
    canPurchaseTicket() {
      if (!this.event || !this.event.eventStartDate) return false;
      const now = new Date();
      const eventDate = new Date(this.event.eventStartDate);
      const diffTime = eventDate.getTime() - now.getTime();
      const diffDays = diffTime / (1000 * 3600 * 24);
      return diffDays <= 7 && diffDays >= 0;
    },
  },
};
</script>

<style scoped>
.event-container {
  width: 100vw;
  min-height: 100vh;
  background: linear-gradient(to right, #2f3132, #c6cbcf);
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.event-header {
  text-align: center;
  background: linear-gradient(135deg, #c12c07, #feb47b);
  color: white;
  padding: 20px;
  width: 90%;
  border-radius: 15px;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.3);
}

.event-title {
  font-size: 32px;
  font-weight: bold;
  text-transform: uppercase;
}
th {
  color: black !important;
}

.tags-container {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 10px;
}

.event-tag,
.event-age-tag {
  background: rgba(255, 255, 255, 0.3);
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 14px;
}

.stars {
  margin-top: 10px;
  display: flex;
  justify-content: center;
  font-size: 22px;
}

.full-star {
  color: gold;
}
.half-star {
  color: gold;
  opacity: 0.6;
}
.empty-star {
  color: gray;
}

.event-content {
  display: flex;
  width: 90%;
  margin-top: 20px;
  gap: 20px;
}

.event-image-carousel {
  flex: 1;
  border-radius: 10px;
  overflow: hidden;
}

.event-image {
  width: 100%;
  height: auto;
  border-radius: 10px;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.3);
}

.event-description-container {
  text-align: center;
  max-width: 80%;
  margin-top: 20px;
}
.event-descriptions {
}
.event-description {
  font-size: 16px;
  line-height: 1.6;
}

.toggle-btn {
  background: none;
  border: none;
  color: #ffb74d;
  cursor: pointer;
  font-size: 16px;
  margin-top: 10px;
}
.event-tags,
.event-age-tag {
  display: inline-block;
  background: #222;
  color: white;
  padding: 5px 15px;
  margin: 5px;
  font-size: 14px;
  border-radius: 5px;
}

.event-content {
  display: flex;
  gap: 20px;
  margin-top: 20px;
}

.event-details {
  flex: 1;
  padding: 20px;
  border-radius: 8px;
  padding: 15px;
  color: black;
}

.event-details h3 {
  margin-bottom: 10px;
  color: #d32f2f;
}

.event-info p {
  font-size: 16px;
  margin: 10px 0;
}

.event-artists {
  margin-top: 20px;
  text-align: center;
}

.event-artists h3 {
  color: #d32f2f;
}

.artist-item {
  display: inline-block;
  background: #eeeeee;
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
}
.toggle-btn {
  background: none;
  border: none;
  color: blue;
  cursor: pointer;
  display: block;
  margin: 10px auto; /* Căn giữa */
}
.event-tickets {
  margin-top: 20px;
  text-align: center;
}

.ticket-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
}

.book-btn {
  margin-top: 10px;
  padding: 8px 15px;
  background: #d32f2f;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

.book-btn:hover {
  background: #b71c1c;
}

.book-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  background: #ccc;
}

.event-image-carousel {
  flex: 1;
  border-radius: 8px;
}

.event-image-slide img {
  width: 100%;
  height: auto;
  border-radius: 8px;
}

.swiper-button-prev,
.swiper-button-next {
  color: white;
  font-size: 20px;
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  z-index: 10;
  background: rgba(0, 0, 0, 0.7);
  padding: 10px;
  border-radius: 50%;
}

.swiper-button-prev {
  left: 10px;
}

.swiper-button-next {
  right: 10px;
}

.swiper-button-prev:hover,
.swiper-button-next:hover {
  background: rgba(0, 0, 0, 0.9);
}
.stars {
  margin-top: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.star {
  font-size: 20px;
  color: #ffcc00;
  margin-right: 3px;
}

.full-star {
  color: #ffcc00;
}

.half-star {
  position: relative;
}

.empty-star {
  color: #e0e0e0;
}

.event-description {
  margin-top: 20px;
  text-align: justify;
  white-space: pre-line;
}

.toggle-btn {
  background: none;
  border: none;
  color: #ffcc00;
  cursor: pointer;
  display: block;
  margin: 10px auto;
}

.event-tickets {
  margin-top: 20px;
  text-align: center;
}

.ticket-list {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  justify-content: center;
}

.ticket-card {
  padding: 15px;
  border-radius: 5px;
  text-align: center;
  background: #cfe2cd;
  color: black;
}

.book-btn {
  margin-top: 10px;
  padding: 8px 15px;
  background: #ffcc00;
  color: black;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  font-weight: bold;
}

.book-btn:hover {
  background: #e6b800;
}
</style>
