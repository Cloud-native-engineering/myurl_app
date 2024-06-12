<template>
  <div class="container">
    <div class="row align-items-center profile-header">
      <div class="col-md-2 mb-3">
        <img
          :src="user?.picture"
          alt="User's profile picture"
          class="rounded-circle img-fluid profile-picture"
        />
      </div>
      <div class="col-md text-center text-md-left">
        <h2>{{ apiUser?.username || user?.name }}</h2>
        <p class="lead text-muted">{{ user?.email }}</p>
      </div>
    </div>

    <div class="row">
      <div class="col-md-12">
        <h3>User Details from API</h3>
        <highlightjs language="json" :code="JSON.stringify(apiUser, null, 2)" />
      </div>
    </div>

    <div class="row">
      <h3>User Details from Auth0 (Token)</h3>
      <highlightjs language="json" :code="JSON.stringify(user, null, 2)" />
    </div>
  </div>
</template>

<script lang="ts">
import { useAuth0 } from '@auth0/auth0-vue';
import { ref, onMounted } from 'vue';

export default {
  name: "profile-view",
  setup() {
    const auth0 = useAuth0();
    const apiUser = ref(null);

    async function callApi() {
      const accessToken = await auth0.getAccessTokenSilently();
      try {
        const response = await fetch("http://localhost:5005/users/me", {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const data = await response.json();
        apiUser.value = data; // Store the data in apiUser.value
      } catch (e: any) {
        apiUser.value = `Error: the server responded with '${e.response.status}: ${e.response.statusText}'`; // Store the error message in apiUser.value
      }
    }

    onMounted(callApi);
    return {
      user: auth0.user,
      apiUser,
    }
  }
};
</script>