<template>
  <div>
    <!-- Header with Title -->
    <header class="site-header">
      <h1>Bike Image Gallery</h1>
    </header>

    <!-- Upload Button -->
    <div class="upload-button">
      <button @click="showUploadForm = !showUploadForm">
        {{ showUploadForm ? 'Hide Upload Form' : 'Upload an Image' }}
      </button>
    </div>

    <!-- Image Upload Form -->
    <div v-if="showUploadForm" class="image-upload">
      <input type="text" v-model="newImageUrl" placeholder="Image URL" />
      <input type="text" v-model="newImageAlt" placeholder="Image Description" />
      <input type="text" v-model="newImageTitle" placeholder="Image Title" />
      <input type="date" v-model="newImageDate" placeholder="Image Date" />
      <button @click="addImage">Add Image</button>
    </div>

    <!-- Upload Status -->
    <div v-if="uploadStatus" class="upload-status">
      <p>{{ uploadStatus }}<span v-if="uploadProgress !== null"> ({{ uploadProgress }}%)</span></p>
    </div>

    <!-- Display Added Image Title -->
    <div v-if="uploadedImageTitle" class="added-image-title">
      <p>Added Image Title: {{ uploadedImageTitle }}</p>
    </div>

    <!-- Image Gallery -->
    <div class="gallery-grid">
      <div v-for="image in images" :key="image.id" class="image-card">
        <img :src="image.src" :alt="image.alt" class="image-card__img" @click="openModal(image)" />
        <button class="delete-btn" @click="deleteImage(image.id)">Delete</button>
      </div>
    </div>

    <!-- Modal -->
    <div v-if="selectedImage" class="modal" @click="closeModal">
      <div class="modal-content" @click.stop>
        <button class="nav-btn prev-btn" @click="prevImage" :disabled="isFirstImage">Previous</button>
        <img :src="selectedImage.src" :alt="selectedImage.alt" class="modal-img" />
        <h2>{{ selectedImage.title }}</h2>
        <p>{{ selectedImage.alt }}</p>
        <p><strong>Date:</strong> {{ selectedImage.date }}</p>
        <button class="nav-btn next-btn" @click="nextImage" :disabled="isLastImage">Next</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      images: [],
      selectedImage: null,
      newImageUrl: '',
      newImageAlt: '',
      newImageTitle: '',
      newImageDate: '',
      uploadStatus: '',
      uploadProgress: null,
      uploadedImageTitle: '', // Store the title of the uploaded image
      showUploadForm: false, // Track whether the upload form is visible
    };
  },
  computed: {
    isFirstImage() {
      return this.selectedImage && this.images.findIndex(image => image.id === this.selectedImage.id) === 0;
    },
    isLastImage() {
      return this.selectedImage && this.images.findIndex(image => image.id === this.selectedImage.id) === this.images.length - 1;
    },
  },
  methods: {
    addImage() {
      if (this.newImageUrl && this.newImageAlt && this.newImageTitle && this.newImageDate) {
        this.uploadStatus = 'Uploading image...';
        this.uploadProgress = 0;

        const interval = setInterval(() => {
          if (this.uploadProgress < 100) {
            this.uploadProgress += 10;
          } else {
            clearInterval(interval);
          }
        }, 100);

        setTimeout(() => {
          const newImage = {
            id: this.images.length + 1,
            src: this.newImageUrl,
            alt: this.newImageAlt,
            title: this.newImageTitle,
            date: this.newImageDate,
          };
          this.images.push(newImage);
          this.uploadStatus = 'Image uploaded successfully!';
          this.uploadedImageTitle = this.newImageTitle; // Store the title of the uploaded image
          this.newImageUrl = '';
          this.newImageAlt = '';
          this.newImageTitle = '';
          this.newImageDate = '';
          this.uploadProgress = null;
          this.showUploadForm = false; // Hide the form after submission
          setTimeout(() => {
            this.uploadStatus = '';
            this.uploadedImageTitle = ''; // Clear the uploaded image title
          }, 2000);
        }, 1000);
      } else {
        this.uploadStatus = 'Please fill in all fields.';
        setTimeout(() => {
          this.uploadStatus = '';
        }, 2000);
      }
    },
    deleteImage(imageId) {
      this.images = this.images.filter(image => image.id !== imageId);
      if (this.selectedImage && this.selectedImage.id === imageId) {
        this.closeModal();
      }
    },
    openModal(image) {
      this.selectedImage = image;
    },
    closeModal() {
      this.selectedImage = null;
    },
    prevImage() {
      const currentIndex = this.images.findIndex(image => image.id === this.selectedImage.id);
      if (currentIndex > 0) {
        this.selectedImage = this.images[currentIndex - 1];
      }
    },
    nextImage() {
      const currentIndex = this.images.findIndex(image => image.id === this.selectedImage.id);
      if (currentIndex < this.images.length - 1) {
        this.selectedImage = this.images[currentIndex + 1];
      }
    },
  },
};
</script>

<style scoped>
body {
  background-color: #2e2e2e; /* Dark background */
  color: #f0f0f0; /* Light text color for better contrast */
  font-family: Arial, sans-serif;
}

.site-header {
  text-align: center;
  margin-bottom: 20px;
  background-color: #3c3c3c; /* Slightly darker grey for header */
  padding: 10px;
  border-bottom: 1px solid #666; /* Subtle bottom border */
}

.site-header h1 {
  color: #ffffff; /* White text */
  font-size: 2.5em;
  font-weight: bold;
}

.upload-button {
  text-align: center;
  margin-bottom: 20px;
}

.upload-button button {
  background-color: #4a90e2; /* Blue button background */
  color: #fff;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
}

.image-upload {
  text-align: center;
  margin-bottom: 20px;
}

.upload-status {
  text-align: center;
  margin-bottom: 20px;
  font-weight: bold;
}

.added-image-title {
  text-align: center;
  margin-bottom: 20px;
}

.gallery-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center; /* Center the images */
}

.image-card {
  position: relative;
}

.image-card__img {
  width: 150px;
  height: 150px;
  object-fit: cover;
  cursor: pointer;
  border: 1px solid #444; /* Border for better separation */
}

.delete-btn {
  position: absolute;
  top: 5px;
  right: 5px;
  background: red;
  color: white;
  border: none;
  cursor: pointer;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: #3c3c3c;
  padding: 20px;
  position: relative;
  color: #fff;
}

.nav-btn {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px;
  cursor: pointer;
  margin: 5px;
}

.prev-btn {
  float: left;
}

.next-btn {
  float: right;
}

.modal-img {
  max-width: 100%;
  max-height: 400px;
  display: block;
  margin: 0 auto;
}

.nav-btn:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>
