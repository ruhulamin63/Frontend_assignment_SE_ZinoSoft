<template>
    <div class="content-wrapper">
        <loading :active="isLoading" :can-cancel="true" :color="'#1C570EAE'" :on-cancel="() => (isLoading = false)"
            :is-full-page="fullPage">
        </loading>

        <!-- Content Header (Page header) -->
        <div class="content-header">
            <div class="container-fluid">
                <div class="row mb-2">
                    <div class="col-sm-6">
                        <h1 class="m-0">Category</h1>
                    </div>
                    <!-- /.col -->
                    <div class="col-sm-6">
                        <ol class="breadcrumb float-sm-right">
                            <li class="breadcrumb-item">
                                <NuxtLink to="/">Home</NuxtLink>
                            </li>
                            <li class="breadcrumb-item active">Category</li>
                        </ol>
                    </div>
                    <!-- /.col -->
                </div>
                <!-- /.row -->
            </div>
            <!-- /.container-fluid -->
        </div>
        <!-- /.content-header -->

        <!-- Main content -->
        <section class="content">
            <div class="container-fluid">
                <div class="row">
                    <div class="col-md-12">
                        <div class="card">
                            <div class="card-header">
                                <h3 class="card-title">Category List</h3>
                                <button style="float: right" type="button" class="add-button" data-toggle="modal"
                                    data-target="#staticBackdrop" @click="openModal">
                                    <i class="fa fa-plus"></i> Add New
                                </button>
                            </div>
                            <!-- table component start -->
                            <common-table :columns="table_header" :entities="data" @getData="getData"
                                :isPagination="true" filter_type="filter">
                                <template v-slot:column_sl="{ index }">
                                    <td>{{ index + 1 }}</td>
                                </template>

                                <template v-slot:column_image="{ entity }">
                                    <td> 
                                        <a :href="entity?.image" target="_blank">
                                            <img :src="base_url + entity?.image" alt="image" style="width: 50px; height: 50px;">
                                        </a>
                                    </td>
                                </template>

                                <template v-slot:column_action="{ entity }">
                                    <td>
                                        <span>
                                            <a href="javascript:" @click="onEdit(entity)"><i
                                                    class="fas fa-edit"></i></a>

                                            &nbsp;

                                            <a style="color: #dc3545" href="javascript:" @click="onDelete(entity)"><i
                                                    class="fas fa-trash"></i></a>
                                        </span>
                                    </td>
                                </template>
                            </common-table>
                            <!-- table component end -->
                        </div>
                    </div>
                </div>
            </div>
            <!-- /.container-fluid -->
        </section>
        <!-- /.content -->

        <!-- modal start -->
        <div class="modal fade" id="staticBackdrop" data-backdrop="static" data-keyboard="false" tabindex="-1"
            aria-labelledby="staticBackdropLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered modal-dialog-scrollable">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="staticBackdropLabel">New Category Create</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <!-- modal start -->
                    <Form @submit="onSubmit" :validation-schema="schema" :initial-values="formData">
                        <div class="modal-body">
                            <div class="row">

                                <div class="col-12">
                                    <div class="form-group">
                                        <label for="name">Name</label>
                                        <Field type="text" class="form-control" id="name" placeholder="Enter name"
                                            name="name" />
                                        <ErrorMessage class="text-danger" name="name" />
                                    </div>
                                </div>

                                <div class="col-12">
                                    <div class="form-group">
                                        <label for="image">Image (<span class="text-danger">format: png, jpg,
                                                jpeg</span>),
                                            (<span class="text-danger">max-size: 2MB</span>)</label>
                                        <input id="selectedImage" class="form-control" type="file" name="file"
                                            @change="uploadImage" accept=".png, .jpg, .jpeg" />

                                        <span class="text-danger">
                                            {{ image_file_error ? image_file_size : "" }}
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="card-footer mt-2">
                            <button :disabled="submitDisabled" type="submit" class="add-button float-right">
                                <i v-if="submitDisabled" class="fa fa-spinner fa-spin"></i>
                                {{ submitDisabled ? "Loading..." : "Submit" }}
                            </button>
                            <button class="btn btn-danger btn-sm text-white float-right mr-1" @click="closeModal"
                                type="button" data-coreui-dismiss="modal">
                                Close
                            </button>
                        </div>
                    </Form>
                    <!-- modal end -->
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
useHead({
    title: "Category",
    meta: [
        {
            name: "description",
            content: "Categories page description",
        },
    ],
});

import { ref, onMounted } from "vue";
import { useToast } from "vue-toastification";
import { Field, Form, ErrorMessage } from "vee-validate";
import * as yup from "yup";

import Loading from "vue-loading-overlay";
import "vue-loading-overlay/dist/css/index.css";
const loading = Loading;
const isLoading = ref(false);
const fullPage = ref(true);

const config = useRuntimeConfig();

const schema = yup.object().shape({
    name: yup.string().min(3).required().label("Name"),
});

const api_url = useRuntimeConfig().public["apiBaseUrl"];
const base_url = useRuntimeConfig().public["baseUrl"];

const table_header = [
    { label: "Sl", field: "sl", sortable: false },
    { label: "Title", field: "name" },
    { label: "Image", field: "image" },
];

let isEdit = ref(false);
let data = ref([]);
const toast = useToast();

const submitDisabled = ref(false);
const image_file = ref("");
const selectedImage = ref(null);
const image_file_error = ref(false);
const image_file_size = ref("");

const formData = ref({
    id: 0,
    name: "",
});

const uploadImage = (event) => {
    const selectImage = event.target.files[0];
    const file_size = selectImage.size; // get the size (bytes) of the file
    const fileName = selectImage.name; // Get the name of the file

    const allowedExtensions = ["png", "jpg", "jpeg"];
    const fileExtension = fileName.split(".").pop().toLowerCase();

    if (!allowedExtensions.includes(fileExtension)) {
        useToast().warning("Only accept png, jpg, jpeg file!");
        selectedImage.value = null;
        event.target.value = "";
        image_file_size.value = "";

        return false;
    } else if (file_size > 2097152) {
        image_file_error.value = true;
        image_file_size.value = "File size must be less than 2 MB";
        image_file.value = selectImage;
        return false;
    }

    image_file.value = selectImage;
    image_file_error.value = false;
    image_file_size.value = "";
};

const resetForm = () => {
    formData.value = {
        id: 0,
        name: "",
    };
};

const showModal = ref(false);

const openModal = () => {
    showModal.value = true;
};

const closeModal = () => {
    $("#staticBackdrop").modal("hide");
    resetForm();
    showModal.value = false;
};

const onSubmit = async (values, { resetForm }) => {
    try {
        submitDisabled.value = true;
        let url = api_url + "categories";
        let method = "POST";

        let formData = new FormData();
        formData.append("name", values.name);
        formData.append("image", image_file.value);

        if (values.id > 0) {
            url = url + "/" + values.id;
            method = "PUT";
        }

        const res = await $fetch(url, {
            method,
            body: formData,
        });
        if (res.status === 200) {
            toast.success(res.message);
            closeModal();
            getData();
            resetForm();
            submitDisabled.value = false;
        } else {
            toast.error(res.message);
            submitDisabled.value = false;
        }
    } catch (error) {
        toast.error("Something went wrong!");
        console.log("submit error", error);
        submitDisabled.value = false;
    }
};

// get data
const getData = async (params = { url: null, filter: { rows: 10 } }) => {
    isLoading.value = true;
    try {
        let url = api_url + "categories";
        if (params.hasOwnProperty("url") && params.url != null) {
            url = params.url;
        }

        let filter = {};
        if (params.hasOwnProperty("filter")) {
            filter = params.filter;
        }

        const response = await $fetch(url, {
            params: filter,
        });
        data.value = response;

    } catch (error) {
        console.log(error);
    }
    isLoading.value = false;
};

onMounted(() => getData());

function onEdit(entity) {
    formData.value = {
        id: entity?.id,
      name: entity?.name,
    };
    $("#staticBackdrop").modal("show");
}

const onDelete = async (entity) => {
    try {
        const res = await $fetch(api_url + "categories/" + entity?.id, {
            method: "DELETE",
        });
        if (res.status === 200) {
            toast.success(res.message);
            getData();
        } else {
            toast.error(res.message);
        }
    } catch (error) {
        console.log("🚀 ~ file: index.vue:217 ~ onDelete ~ error:", error);
    }
};
</script>

<style scoped></style>