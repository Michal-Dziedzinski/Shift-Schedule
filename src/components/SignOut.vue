<template>
  <v-btn color="orange-darken-1" @click="handleSignOut" append-icon="mdi-logout"
    >Wyloguj</v-btn
  >
</template>

<script setup lang="ts">
import { supabase } from "../supabase";

const handleSignOut = async () => {
  const { error } = await supabase.auth.signOut();
  if (error) {
    console.error("Error signing out:", error.message);
  }
};
</script>

<style lang="scss" scoped></style>
