<template>
  <section class="w-full flex justify-center lg:items-center lg:h-screen">
    <BaseForm
      :title="titlePage"
      :register="registerClient"
      :update="updateClient"
      :disabledSendBtn="disabledSendBtn"
      :loadingRequest="isLoading"
    >
      <template v-slot:form>
        <div class="flex flex-col justify-start items-start">
          <label for="" class="test-start">Nome</label>
          <input
            type="text"
            class="input-text"
            placeholder="Informe o nome do cliente"
            v-model="name"
            :disabled="isLoading"
          />
        </div>

        <div class="flex flex-col justify-start items-start mt-5">
          <label for="" class="test-start">Email</label>
          <input
            type="text"
            class="input-text"
            placeholder="Informe seu email"
            v-model="email"
            :disabled="isLoading"
          />
        </div>

        <div class="flex flex-col justify-start items-start mt-5">
          <label for="" class="test-start">Cidade</label>
          <input
            type="text"
            class="input-text"
            placeholder="Informe a cidade"
            v-model="city"
            :disabled="isLoading"
          />
        </div>

        <Dropzone
          titleDropzone="Arraste ou clique aqui para importar imagem do produto"
          :files="listFilesUpload"
          @onFileUpload="($event) => (listFilesUpload = $event)"
          :isLoading="isLoading"
        />
      </template>
    </BaseForm>
  </section>
</template>

<script setup>
import { computed, onMounted, ref } from "vue";
import BaseForm from "@/components/BaseForm/Main.vue";
import api from "@/services/Api";
import { useRoute, useRouter } from "vue-router";
import Dropzone from "@/components/Dropzone/Main.vue";
import toast from "@/services/Toast";
// import { getExtnsionFile, urlToFile } from "@/services/Helper";

const route = useRoute();
const router = useRouter();
const idUpdate = computed(() => route.params?.id);
const isLoading = ref(false);
const name = ref("");
const email = ref("");
const city = ref("");

const listFilesUpload = ref([]);

onMounted(() => {
  if (idUpdate.value != "novo") {
    getClientBy();
  }
});

const disabledSendBtn = computed(() => name.value == "" || city.value == "" || email.value == "");
const titlePage = computed(() =>
  idUpdate.value ? "Editar cliente" : "Cadastrar novo cliente"
);

const registerClient = async () => {
  try {
    isLoading.value = true;

    const { status, data } = await api.post("/clients", {
      name: name.value,
      email: email.value,
      city: city.value,
    });

    if (status == 201) {
      if (listFilesUpload.value.length > 0) {
        const status = await sendImageClient(data.id);

        if (status == 200) {
          toast.success("Cadastrado com sucesso!", {
            autoClose: 2500,
          });

          name.value = "";
          city.value = "";

          return setTimeout(() => {
            router.back();
          }, 2500);
        }
      }

      await removeImageClient(data.id);

      toast.success("Cadastrado com sucesso!", {
        autoClose: 2500,
      });

      name.value = "";
      city.value = "";

      return setTimeout(() => {
        router.back();
      }, 2500);
    }
  } catch (err) {
    if (err?.response && err?.response?.data) {
      err.response.data.errors.map((error) => toast.error(error.message));
      return;
    }

    return toast.error("Algo deu errado. Tente novamente.");
  } finally {
    isLoading.value = false;
  }
};

const sendImageClient = async (id) => {
  const formData = new FormData();

  listFilesUpload.value.forEach((item) => {
    formData.append("File", item);
  });

  formData.append("Id", id);

  try {
    const { status } = await api.post("/clients/file", formData);
    return status;
  } catch (err) {
    console.log(err);
  }
};

const removeImageClient = async (id) => {
  try {
    await api.delete(`/clients/file/${id}`);
  } catch (err) {
    console.log(err);
  }
};

const updateClient = async () => {
  try {
    isLoading.value = true;

    const { status } = await api.put("/clients", {
      id: idUpdate.value,
      name: name.value,
      email: email.value,
      city: city.value,
    });

    if (status == 200) {
      if (listFilesUpload.value.length > 0) {
        try {
          const status = await sendImageClient(idUpdate.value);

          if (status == 200) {
            toast.success("Atualizado com sucesso!", {
              autoClose: 2500,
            });

            name.value = "";
            city.value = "";

            return setTimeout(() => {
              router.back();
            }, 2500);
          }
        } catch (err) {
          if (err?.response && err?.response?.data) {
            err.response.data.errors.map((error) => toast.error(error.message));
            return;
          }

          return toast.error("Algo deu errado. Tente novamente.");
        }
      }

      await removeImageClient(idUpdate.value);

      toast.success("Atualizado com sucesso!", {
        autoClose: 2500,
      });

      name.value = "";
      city.value = "";

      return setTimeout(() => {
        router.back();
      }, 2500);
    }
  } catch (err) {
    if (err?.response && err?.response?.data) {
      err.response.data.errors.map((error) => toast.error(error.message));
      return;
    }

    return toast.error("Algo deu errado. Tente novamente.");
  } finally {
    isLoading.value = false;
  }
};

const getClientBy = async () => {
  try {
    isLoading.value = true;

    const { data } = await api.get(`/clients/${idUpdate.value}`);

    if (data) {
      name.value = data.name;
      email.value = data.email;
      city.value = data.city;
      email.value = data.email;
    }
  } catch (err) {
    if (err?.response && err?.response?.data) {
      err.response.data.errors.map((error) => toast.error(error.message));
      return;
    }

    return toast.error("Algo deu errado. Tente novamente.");
  } finally {
    isLoading.value = false;
  }
};
</script>

<style lang="scss" scoped></style>
