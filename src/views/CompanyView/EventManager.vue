<template>
  <div class="eventList container mx-auto p-6">
    <h1 class="text-2xl font-bold text-center mb-6">Quản lý Sự Kiện</h1>

    <div v-if="loading" class="text-center text-gray-500">
      Loading events...
    </div>

    <div v-if="events.length > 0">
      <!-- Container for event items -->
      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6"
      >
        <div
          v-for="event in events"
          :key="event.eventId"
          @click="goToEventDetails(event.eventId)"
          class="bg-white shadow-lg rounded-lg overflow-hidden p-4 transition transform hover:scale-105 cursor-pointer"
        >
          <h3 class="text-lg font-semibold mb-2">{{ event.eventTitle }}</h3>
          <p class="text-gray-600 mb-2 event-description">
            {{ event.eventDescription }}
          </p>

          <p class="text-sm">
            <strong>Ngày Bắt đầu:</strong>
            {{ formatDate(event.eventStartDate) }}
          </p>
          <p class="text-sm">
            <strong>Ngày Kết Thúc:</strong> {{ formatDate(event.eventEndDate) }}
          </p>
          <p class="text-sm">
            <strong>Giá:</strong> {{ formatCurrency(event.eventPrice) }}
          </p>
          <p class="text-sm">
            <strong>Trạng thái:</strong> {{ event.eventStatus }}
          </p>

          <!-- Display Stars -->
          <div class="flex items-center mt-2 text-yellow-500">
            <span
              v-for="index in getStars(event).fullStars"
              :key="'full-' + index"
              class="text-xl"
            >
              ★
            </span>
            <span v-if="getStars(event).halfStars" class="text-xl">★</span>
            <span
              v-for="index in getStars(event).emptyStars"
              :key="'empty-' + index"
              class="text-xl text-gray-300"
            >
              ★
            </span>
          </div>

          <!-- Event Image -->
          <img
            :src="getPosterImage(event.eventListImgURL)"
            alt="Event Image"
            class="w-full h-40 object-cover mt-3 rounded-md"
          />
        </div>
      </div>

      <!-- Pagination Controls -->
      <div class="flex justify-between items-center mt-6">
        <button
          @click="prevPage"
          :disabled="currentPage === 1"
          class="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 disabled:opacity-50"
        >
          <i class="fas fa-chevron-left"></i> Trang trước
        </button>
        <span class="font-semibold"
          >Trang {{ currentPage }} / {{ totalPages }}</span
        >
        <button
          @click="nextPage"
          :disabled="currentPage >= totalPages"
          class="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 disabled:opacity-50"
        >
          Trang sau <i class="fas fa-chevron-right"></i>
        </button>
      </div>
    </div>

    <div v-if="events.length === 0" class="text-center text-gray-500 mt-6">
      No events found.
    </div>
  </div>
</template>

<script>
import { api } from "@/api/Api";
import { formatCurrency } from "@/composable/format";
export default {
  data() {
    return {
      events: [], // Store events data
      loading: false, // Loading flag
      currentPage: 1,
      itemsPerPage: 12,
      totalPages: 1,
      totalElements: 0,
    };
  },
  mounted() {
    this.getEvents();
  },
  computed: {
    paginatedAccounts() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.events.slice(start, end);
    },
  },
  methods: {
    formatCurrency,
    getPosterImage(imgURLs) {
      // Tìm ảnh có tên poster.jpg, nếu không tìm thấy, trả về phần tử đầu tiên trong mảng
      return imgURLs.find((url) => url.includes("poster.jpg")) || imgURLs[0];
    },
    // Fetch events data from the API
    async getEvents() {
      this.loading = true;
      try {
        const response = await api.get(
          `/events/filter?page=${this.currentPage - 1}&size=${
            this.itemsPerPage
          }`
        );

        if (response.data.status === "OK") {
          this.events = response.data.data.content; // Store events
          this.totalPages = response.data.data.totalPages; // Update total pages
        } else {
          this.$toast.error(
            error.response?.data?.message || "Error fetching events:"
          );
          console.error("Error fetching events:", response.data.message);
        }
      } catch (error) {
        console.error("Error fetching events:", error);
        this.$toast.error(
          error.response?.data?.message || "Error fetching events:"
        );
      } finally {
        this.loading = false;
      }
    },

    // Change page and fetch new data
    changePage(pageNumber) {
      if (pageNumber > 0 && pageNumber <= this.totalPages) {
        this.currentPage = pageNumber;
        this.getEvents();
      }
    },

    // Method to format date
    formatDate(dateString) {
      const date = new Date(dateString);
      return date.toLocaleString();
    },

    // Calculate average rating for a single event
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
    goToEventDetails(eventId) {
      this.$router.push({ path: `/company/event/${eventId}` });
    },

    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
        this.getEvents();
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
        this.getEvents();
      }
    },
  },
};
</script>

<style scoped>
/* Tổng thể */
body {
  font-family: "Roboto", sans-serif;
  background-color: #f7f8fa;
  margin: 0;
  padding: 0;
}

/* Thẻ sự kiện */
.event {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin-right: 20px; /* Thêm margin phải để có khoảng cách giữa các thẻ */
  margin-bottom: 20px;
  transition: all 0.3s ease-in-out;
  display: flex;
  flex-direction: column;
  max-width: 300px; /* Giới hạn chiều rộng tối đa */
  width: 100%;
}

.event:hover {
  transform: translateY(-10px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
}
.event-description {
  display: -webkit-box;
  -webkit-line-clamp: 3; /* Hiển thị tối đa 3 dòng */
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis; /* Hiển thị dấu "..." nếu quá dài */
  max-width: 100%;
}
.event h3 {
  font-size: 1.6rem;
  font-weight: 600;
  color: #333;
  margin-bottom: 10px;
}

.event p {
  font-size: 1rem;
  color: #666;
  line-height: 1.5;
}

.event img {
  width: 100%;
  height: auto;
  border-radius: 8px;
  margin-top: 15px;
  transition: transform 0.3s ease-in-out;
}

.event img:hover {
  transform: scale(1.05);
}
.event-description {
  display: -webkit-box;
  /* -webkit-line-clamp: 3; Hiển thị tối đa 3 dòng */
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis; /* Hiển thị dấu "..." nếu quá dài */
  max-width: 100%;
}

/* Đánh giá sao */
.stars {
  margin-top: 10px;
  display: flex;
  align-items: center;
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

/* Pagination */
.pagination {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.pagination button {
  background-color: #ff6347;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s, transform 0.3s ease;
  margin: 0 8px;
}

.pagination button:hover {
  background-color: #ff4500;
  transform: scale(1.05);
}

.pagination button:disabled {
  background-color: #ddd;
  cursor: not-allowed;
}

.pagination span {
  padding: 10px;
  font-size: 16px;
  color: #333;
  font-weight: 500;
}

/* Thiết kế cấu trúc trang (chuyển sang layout ngang) */
.event-list {
  display: flex;
  flex-wrap: wrap; /* Allow wrapping for small screens */
  gap: 20px;
  justify-content: flex-start;
}

.event {
  width: 280px; /* Điều chỉnh lại chiều rộng của các sự kiện */
}

/* Pagination control should be aligned with the horizontal layout */
.pagination {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

/* Trang tải và không có sự kiện */
.loading {
  text-align: center;
  font-size: 1.2rem;
  color: #ff6347;
  margin-top: 30px;
}

.no-events {
  text-align: center;
  font-size: 1.2rem;
  color: #ff6347;
  font-weight: bold;
  margin-top: 30px;
}

/* Responsive design */
@media (max-width: 768px) {
  .event-list {
    flex-direction: column; /* Chuyển sang dọc khi màn hình nhỏ */
    align-items: center;
  }

  .pagination {
    flex-direction: column;
  }
}

@media (max-width: 480px) {
  .event {
    width: 100%; /* Thẻ sự kiện chiếm toàn bộ chiều rộng */
    margin-bottom: 20px;
  }

  .pagination button {
    padding: 8px 15px;
  }
}
</style>
