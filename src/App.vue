<template>
  <button data-bs-toggle="modal" data-bs-target="#exampleModal" data-bs-whatever="@mdo" class="btn btn-success" style="margin: 1.5rem" @click="openModal">Add</button>
  <div v-for="(category, index) in categorys" :key="index">
    <div class="card mb-3" style="max-width: 540px; margin: auto; margin-top: 2rem;">
  <div class="row g-0">
    <div class="col-md-4">
      <img :src="category.image" class="img-fluid rounded-start" alt="category-image">
    </div>
    <div class="col-md-8">
      <div class="card-body">
        <h5 class="card-title">{{category.nameEn}}</h5>
        <p class="card-text">This is a wider card with supporting text below as a natural lead-in to additional content. This content is a little bit longer.</p>
        <p class="card-text"><small class="text-muted">{{ category.isActive === false ? 'Out of stock' : 'In stock'}}</small></p>
        <button data-bs-toggle="modal" data-bs-target="#exampleModal" data-bs-whatever="@mdo" style="margin-right: 5px;" @click="editCategory(category)" class="btn btn-success">Edit</button>
        <button @click="deleteCategory(category.code)" class="btn btn-danger">Delete</button>
      </div>
    </div>
  </div>
</div>
  </div>

   <!-- Form-Modal -->
  <div>
    <div class="modal fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h1 class="modal-title fs-5" id="exampleModalLabel">Frequently Asked Questions</h1>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="submitForm">
              <section class="d-flex gap-4">
              <div class="mb-3 w-100">
                <label class="col-form-label">Code</label>
                <input type="text" class="form-control" v-model="formCategory.code" placeholder="Code" :disabled="isEdit">
              </div>
              <div class="mb-3 w-100">
                <label class="col-form-label">NameEn</label>
                <input type="text" class="form-control" v-model="formCategory.nameEn" placeholder="Input Name for EN">
              </div>
            </section>
            <section class="d-flex gap-4">
            <div class="mb-3 w-100">
              <label class="col-form-label">NameKh</label>
              <input type="text" class="form-control" v-model="formCategory.nameKh" placeholder="Input Name for EN" required>
            </div>
            <div class="mb-3 w-100">
                <label class="col-form-label">Image</label>
                <input type="file" class="form-control" @change="handleImageUpload" accept="image/*"  ref="fileImageInput">
              <div v-if="imagePreview">
                <img :src="imagePreview" class="mt-2" style="max-width: 100px;" alt="category-image">
              </div>
              </div>
            </section>
            <section class="d-flex gap-4">
              <div class="mb-3 w-100">
                <label class="col-form-label">Active</label>
                <select class="form-select" aria-label="Default select example" v-model="formCategory.isActive">
                  <option :value="true">In Stock</option>
                  <option :value="false">Out of stock</option>
                </select>
              </div>
              <div class="mb-3 w-100">
                <label class="col-form-label">Display Order</label>
                <input type="text" class="form-control" v-model="formCategory.displayOrder" placeholder="DisplayOrder">
              </div>
            </section>
              <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                <button type="submit" data-bs-dismiss="modal" class="btn btn-primary">{{ isEdit ? 'Submit' : 'Add' }}</button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import axios from 'axios';
import { onMounted, ref } from 'vue';
interface Category {
  code: number | null;
  nameEn?: string;  
  nameKh?: string;
  image?: string;
  isActive?: boolean;
  displayOrder?: number | null;
}
const categorys = ref<Category[]>([])
const isEdit = ref(false);
const Base_Api = 'https://api-dev-supermarket.chipmongretail.com/api/v1/Category'
const authorizationToken  = 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1bmlxdWVfbmFtZSI6IjkwMDE5MjE3IiwianRpIjoiYmU4ODUwNzctOTQyMS00ODc5LWI3NDYtZDE1ZTg1MDgzZjk1IiwiYXVkIjpbImh0dHBzOi8vYXBpLWRldi1jb21tb24uY2hpcG1vbmdyZXRhaWwuY29tOjgwODEiLCJodHRwczovL2FwaS1kZXYtbWFsbC5jaGlwbW9uZ3JldGFpbC5jb20iLCJodHRwczovL2FwaS1kZXYtc3VwZXJtYXJrZXQuY2hpcG1vbmdyZXRhaWwuY29tIl0sIm5iZiI6MTczMjUxNTQ2NSwiZXhwIjoxNzMzMTIwMjY1LCJpYXQiOjE3MzI1MTU0NjUsImlzcyI6Imh0dHBzOi8vYXBpLWRldi1jb21tb24uY2hpcG1vbmdyZXRhaWwuY29tOjgwODEifQ.5f5H71nGt9IjrrymcEXDJDb-T_bBwdrL9oqraVbYtXQ'
const formCategory = ref<Category>({
  code: null,
  nameEn: '',
  nameKh: '',
  image: '',
  isActive: false,
  displayOrder: null,
});
const selectedFile = ref(null);
const imagePreview = ref('')
const openModal = () =>{
  isEdit.value = false;
  formCategory.value = {
  code: null,
  nameEn: '',
  nameKh: '',
  image: '',
  isActive: false,
  displayOrder: null,
  }
  imagePreview.value = '';  
  clearFilePhotoInput();
}

const fetchCategory = async () =>{
  try {
    const response = await axios.get(`${Base_Api}/GetAll`, {
      headers: {
        Authorization: `${authorizationToken}`
      }
    });
    categorys.value = response.data.data;
  } catch (err) {
    console.log('fetching is error', err);
  }
}

const editCategory = (category: Category) => {
  isEdit.value = true;
  formCategory.value = { ...category }; //... clone all filed from form
  imagePreview.value = category.image || '';
};

const submitForm = async () => {
  try {
    if (isEdit.value) {
      await axios.put(`${Base_Api}/Update/${formCategory.value.code}`, formCategory.value, {
        headers: {
          Authorization: `${authorizationToken}`,
          'Content-Type': 'multipart/form-data'
        } 
      });
    } else {
      await axios.post(`${Base_Api}/Create`, formCategory.value, {
        headers: {
          Authorization: `${authorizationToken}`,
          'Content-Type': 'multipart/form-data'
        }
      });
    }
    fetchCategory();
  } catch (err) { 
    console.log('Error submitting form', err);
  }
};

const deleteCategory = async (code: number | null) =>{
    await axios.delete(`${Base_Api}/Delete/${code}`, {
      headers: {
            Authorization: `${authorizationToken}`
      } 
    })
    fetchCategory();
}

const handleImageUpload = (event: any) => {
      const file = event.target.files[0];
      if (file) {
        selectedFile.value = file;
        imagePreview.value = URL.createObjectURL(file);
        formCategory.value.image = file;
  }
}
const fileImageInput = ref<HTMLInputElement | null>(null);

const clearFilePhotoInput = () => {
  if (fileImageInput.value) {
    fileImageInput.value.value = '';
  }
  formCategory.value.image = '';
  imagePreview.value = ''; 
};

onMounted(() =>{
  fetchCategory()
})
</script>


<style scoped>

</style>  