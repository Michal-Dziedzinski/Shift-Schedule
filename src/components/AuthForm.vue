<template>
  <form @submit.prevent="handleLogin">
    <v-alert
      v-if="showAlert"
      text="Błąd logowania, spróbuj jeszcze raz"
      type="error"
      title="Błąd"
      closable
    ></v-alert>
    <v-text-field label="E-mail" type="email" v-model="email" />
    <v-text-field
      :append-inner-icon="visible ? 'mdi-eye-off' : 'mdi-eye'"
      label="Hasło"
      :type="visible ? 'text' : 'password'"
      v-model="password"
      @click:append-inner="visible = !visible"
    />
    <v-btn type="submit">Zaloguj</v-btn>
  </form>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { supabase } from "../supabase";

const email = ref("");
const password = ref("");
const visible = ref(false);
const showAlert = ref(false);

const handleLogin = async () => {
  showAlert.value = false;
  const { error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  });
  if (error) {
    showAlert.value = true;
    console.error("Error logging in:", error.message);
  }
};
</script>

<style lang="scss" scoped></style>
