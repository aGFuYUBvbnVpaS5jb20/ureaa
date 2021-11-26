<template>
  <v-app-bar
    app
    color="blue accent-3"
    dark
  >
    <v-app-bar-nav-icon @click="$store.state.drawer = true"></v-app-bar-nav-icon>

    <v-toolbar-title>요소수 지도</v-toolbar-title>

    <template v-slot:extension>
      <v-tabs
        :align-with-title="!$vuetify.breakpoint.xsOnly"
        color="white"
        v-model="$store.state.tab"
      >
        <v-tab>Map</v-tab>
      </v-tabs>
    </template>
  </v-app-bar>
</template>
