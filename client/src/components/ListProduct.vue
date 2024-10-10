<script setup>
import { onMounted, reactive, ref } from 'vue';
const listProduct = reactive([]);
const newProduct = reactive({
  id: null, 
  name: '',
  price: '',
  quantity: '',
  image: '',
  date: '',
});
const showForm = ref(false); 
const isEditing = ref(false); 
const getProducts = async () => {
  try {
    const response = await fetch('http://localhost:3000/products');
    const data = await response.json();
    listProduct.push(...data);
  } catch {
    console.log('Lỗi khi lấy danh sách sản phẩm');
  }
};

onMounted(() => {
  getProducts();
});

const handleDelete = (id) => {
  if (confirm('Bạn có muốn xóa sản phẩm này không?')) {
    deleteProductById(id);
  }
};

const deleteProductById = async (productId) => {
  try {
    const index = listProduct.findIndex(product => product.id === productId);
    if (index !== -1) {
      const response = await fetch(`http://localhost:3000/products/${productId}`, {
        method: 'DELETE',
      });

      if (response.ok) {
        console.log(`Sản phẩm có ID ${productId} đã được xóa`);
        listProduct.splice(index, 1);
      } else {
        console.log('Không thể xóa sản phẩm');
      }
    } else {
      console.log('Không tìm thấy sản phẩm để xóa');
    }
  } catch {
    console.error('Lỗi khi xóa sản phẩm');
  }
};
const addProduct = async () => {
  try {
    const response = await fetch('http://localhost:3000/products', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(newProduct),
    });

    if (response.ok) {
      const product = await response.json();
      listProduct.push(product); 
      console.log('Sản phẩm mới đã được thêm:', product);
      resetForm(); 
      showForm.value = false;
    } else {
      console.log('Không thể thêm sản phẩm');
    }
  } catch {
    console.error('Lỗi khi thêm sản phẩm');
  }
};
const editProduct = async () => {
  try {
    const response = await fetch(`http://localhost:3000/products/${newProduct.id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(newProduct),
    });

    if (response.ok) {
      const updatedProduct = await response.json();
      const index = listProduct.findIndex(product => product.id === newProduct.id);
      if (index !== -1) {
        listProduct[index] = updatedProduct;
      }
      console.log('Sản phẩm đã được cập nhật:', updatedProduct);
      resetForm(); 
      showForm.value = false; 
      isEditing.value = false; 
    } else {
      console.log('Không thể cập nhật sản phẩm');
    }
  } catch {
    console.error('Lỗi khi cập nhật sản phẩm');
  }
};
const startEditing = (product) => {
  newProduct.id = product.id;
  newProduct.name = product.name;
  newProduct.price = product.price;
  newProduct.quantity = product.quantity;
  newProduct.image = product.image;
  showForm.value = true;
  isEditing.value = true;
};
const resetForm = () => {
  newProduct.id = null;
  newProduct.name = '';
  newProduct.price = '';
  newProduct.quantity = '';
  newProduct.image = '';
  isEditing.value = false;
};
</script>

<template>
  <div class="product-list">
    <button class="btn-add-product" @click="showForm = !showForm">
      {{ isEditing ? 'Cập nhật sản phẩm' : 'Thêm mới sản phẩm' }}
    </button>

    <!-- Form để thêm/sửa sản phẩm -->
    <div v-if="showForm" class="add-product-form">
      <input v-model="newProduct.name" placeholder="Tên sản phẩm" />
      <input v-model="newProduct.image" placeholder="Hình ảnh (url)" />
      <input v-model="newProduct.price" placeholder="Giá" />
      <input v-model="newProduct.quantity" placeholder="Số lượng" />
      <button @click="isEditing ? editProduct() : addProduct()">
        {{ isEditing ? 'Cập nhật' : 'Lưu' }}
      </button>
      <button @click="showForm = false">Hủy</button>
    </div>

    <table class="product-table">
      <thead>
        <tr>
          <th>#</th>
          <th>Tên sản phẩm</th>
          <th>Hình ảnh</th>
          <th>Giá</th>
          <th>Số lượng (kg)</th>
          <th>Ngày thêm</th>
          <th>Chức năng</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(product, index) in listProduct" :key="index">
          <td>{{ index + 1 }}</td>
          <td>{{ product.name }}</td>
          <td><img :src="product.image" alt="product image" class="product-image"/></td>
          <td>{{ product.price }} đ</td>
          <td>{{ product.quantity }}</td>
          <td>21/02/2023</td>
          <td>
            <button class="btn-edit" @click="startEditing(product)">Sửa</button>
            <button class="btn-delete" @click="handleDelete(product.id)">Xóa</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style>
.product-list {
  width: 80%;
  margin: 20px auto;
}

.btn-add-product {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  margin-bottom: 15px;
  cursor: pointer;
}

.add-product-form {
  margin-bottom: 20px;
}

.product-table {
  width: 100%;
  border-collapse: collapse;
  background-color: #f9f9f9;
}

.product-table th, .product-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
}

.product-table th {
  background-color: #4CAF50;
  color: white;
}

.product-image {
  width: 60px;
  height: 60px;
  object-fit: cover;
}

.btn-edit {
  background-color: #ffc107;
  color: white;
  border: none;
  padding: 5px 10px;
  margin-right: 5px;
  cursor: pointer;
}

.btn-delete {
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}
</style>
