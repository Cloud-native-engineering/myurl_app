<template>
  <div>
    <div class="mb-5">
      <h1>URL</h1>
      <p>
        Manages all the URLs that you have created. You can create new URLs,
        edit existing ones, and delete them.
      </p>

      <button class="btn btn-primary mt-5" @click="callApi">
        <font-awesome-icon icon="arrows-rotate" /> Refresh
      </button>
      <br />
      <br />
      <button class="btn btn-primary" @click="openAddModal">
        <font-awesome-icon icon="plus" /> Create
      </button>
    </div>

    <table class="table table-striped">
      <thead>
        <tr>
          <th>ID</th>
          <th>Enabled</th>
          <th>Verified</th>
          <th>Original URL</th>
          <th>Short URL</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(item, index) in apiMessage"
          :key="index"
          @click="openModal(item)"
        >
          <td>{{ item.id }}</td>
          <td>{{ item.is_enabled }}</td>
          <td>{{ item.is_verified }}</td>
          <td>{{ item.original_url }}</td>
          <td>{{ item.short_url }}</td>
          <td>
            <button class="btn btn-primary" @click.stop="openShareModal(item)">
              <font-awesome-icon :icon="['fas', 'share-nodes']" />
            </button>
            <button class="btn btn-primary" @click.stop="openModal(item)">
              <font-awesome-icon icon="pen-to-square" />
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Add Modal -->
    <div
      v-if="newItem"
      class="modal fade show"
      style="display: block"
      tabindex="-1"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Add URL</h5>
            <button
              type="button"
              class="btn-close"
              @click="closeAddModal"
            ></button>
          </div>
          <div class="modal-body">
            <form>
              <div class="mb-3">
                <label for="original_url" class="form-label"
                  >Original URL</label
                >
                <input
                  type="text"
                  class="form-control"
                  id="original_url"
                  v-model="newItem.original_url"
                />
              </div>
              <div class="mb-3">
                <label for="short_url" class="form-label">Short URL</label>
                <input
                  type="text"
                  class="form-control"
                  id="short_url"
                  v-model="newItem.short_url"
                />
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-secondary"
              @click="closeAddModal"
            >
              <font-awesome-icon icon="xmark" />
            </button>
            <button
              type="button"
              class="btn btn-primary"
              @click="addUrl(newItem)"
            >
              <font-awesome-icon :icon="['fas', 'floppy-disk']" />
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Share Modal -->
    <div
      v-if="shareItem"
      class="modal fade show"
      style="display: block"
      tabindex="-1"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Share URL</h5>
            <button
              type="button"
              class="btn-close"
              @click="closeShareModal"
            ></button>
          </div>
          <div class="modal-body">
            <form>
              <div class="mb-3">
                <label for="username" class="form-label">Username</label>
                <input
                  type="text"
                  class="form-control"
                  id="username"
                  v-model="username"
                />
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-secondary"
              @click="closeShareModal"
            >
              <font-awesome-icon icon="xmark" />
            </button>
            <button
              type="button"
              class="btn btn-primary"
              @click="shareUrl(shareItem.id, username)"
            >
              <font-awesome-icon :icon="['fas', 'floppy-disk']" />
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Edit Modal -->
    <div
      v-if="selectedItem"
      class="modal fade show"
      style="display: block"
      tabindex="-1"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Edit URL</h5>
            <button
              type="button"
              class="btn-close"
              @click="closeModal"
            ></button>
          </div>
          <div class="modal-body">
            <form>
              <div class="mb-3">
                <label for="original-url" class="form-label"
                  >Original URL</label
                >
                <input
                  type="text"
                  class="form-control"
                  id="original-url"
                  v-model="selectedItem.original_url"
                />
              </div>
              <div class="mb-3">
                <label for="short-url" class="form-label">Short URL</label>
                <input
                  type="text"
                  class="form-control"
                  id="short-url"
                  v-model="selectedItem.short_url"
                />
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" @click="closeModal">
              <font-awesome-icon icon="xmark" />
            </button>
            <button
              type="button"
              class="btn btn-primary"
              @click="
                updateUrl(selectedItem.id, {
                  original_url: selectedItem.original_url,
                  short_url: selectedItem.short_url,
                })
              "
            >
              <font-awesome-icon :icon="['fas', 'floppy-disk']" />
            </button>
            <button
              type="button"
              class="btn btn-danger"
              @click="deleteUrl(selectedItem.id)"
            >
              Delete
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { useAuth0 } from "@auth0/auth0-vue";
import { ref, onMounted } from "vue";
import { library } from "@fortawesome/fontawesome-svg-core";
import {
  faArrowsRotate,
  faFloppyDisk,
  faShareNodes,
  faPlus,
  faXmark,
  faPenToSquare,
} from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";

library.add(
  faFloppyDisk,
  faArrowsRotate,
  faShareNodes,
  faPlus,
  faXmark,
  faPenToSquare
);

export default {
  name: "api-view",
  components: {
    FontAwesomeIcon,
  },
  setup() {
    const auth0 = useAuth0();
    const apiMessage = ref([]);
    const selectedItem = ref(null);
    const shareItem = ref(null);
    const username = ref("");
    const newItem = ref(null);

    async function callApi() {
      const accessToken = await auth0.getAccessTokenSilently();
      try {
        const response = await fetch("/api/urls", {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const data = await response.json();
        apiMessage.value = data;
      } catch (e: any) {
        apiMessage.value = `Error: the server responded with '${e.response.status}: ${e.response.statusText}'`;
      }
    }

    async function updateUrl(url_id, updatedData) {
      const accessToken = await auth0.getAccessTokenSilently();
      try {
        const response = await fetch(`/api/urls/${url_id}`, {
          method: "PATCH",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${accessToken}`,
          },
          body: JSON.stringify(updatedData),
        });

        if (!response.ok) {
          const data = await response.json();
          window.alert(`Error updating URL: ${data.error}`); // Display the error message as a popup
          return;
        }

        callApi();
      } catch (e: any) {
        console.error(`Error updating URL: ${e}`);
      }
    }

    async function deleteUrl(url_id) {
      const accessToken = await auth0.getAccessTokenSilently();
      try {
        await fetch(`/api/urls/${url_id}`, {
          method: "DELETE",
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        callApi();
      } catch (e: any) {
        console.error(`Error deleting URL: ${e}`);
      }
    }

    async function shareUrl(url_id, username) {
      const accessToken = await auth0.getAccessTokenSilently();
      try {
        const response = await fetch(
          `/api/urls/${url_id}/share`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${accessToken}`,
            },
            body: JSON.stringify({ username }),
          }
        );

        if (!response.ok) {
          const data = await response.json();
          window.alert(`Error sharing URL: ${data.error}`); // Display the error message as a popup
          return;
        }

        closeShareModal();
      } catch (e: any) {
        console.error(`Error sharing URL: ${e}`);
      }
    }

    async function addUrl(item) {
      try {
        const accessToken = await auth0.getAccessTokenSilently();
        const response = await fetch(`/api/urls`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${accessToken}`,
          },
          body: JSON.stringify(item),
        });

        if (!response.ok) {
          const data = await response.json();
          window.alert(`Error adding URL: ${data.error}`);
          return;
        }

        closeAddModal();
        callApi();
      } catch (e: any) {
        console.error(`Error adding URL: ${e}`);
      }
    }

    function openAddModal() {
      newItem.value = { original_url: "", short_url: "" };
    }

    function closeAddModal() {
      newItem.value = null;
    }

    function openModal(item) {
      selectedItem.value = { ...item };
    }

    function closeModal() {
      selectedItem.value = null;
    }

    function openShareModal(item) {
      shareItem.value = { ...item };
    }

    function closeShareModal() {
      shareItem.value = null;
      username.value = "";
    }

    onMounted(callApi);

    return {
      apiMessage,
      selectedItem,
      callApi,
      shareItem,
      username,
      openShareModal,
      closeShareModal,
      shareUrl,
      updateUrl,
      deleteUrl,
      openModal,
      closeModal,
      newItem,
      openAddModal,
      closeAddModal,
      addUrl,
    };
  },
};
</script>
