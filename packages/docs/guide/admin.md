# Admin

:::tip PROJECT STRUCTURE
Check [directory structure](getting-started#directory-structure) for big picture of all files location that we will talk about after.  
:::

The next piece of code represent the bare minimal code in order to get VtecAdmin working :

**`src/plugins/admin.js`**

```js
import Vue from "vue";
import VtecAdmin from "vtec-admin";

/**
 * Register all third-party components as Portal Vue, Vuedraggable
 * Will automatically load all CRUD pages resources as well
 */
import "vtec-admin/dist/loader";
// Import custom admin CSS
import "vtec-admin/dist/admin.css";

// Load data and auth providers to use with your API
import {
  laravelDataProvider,
  sanctumAuthProvider,
} from "vtec-admin/dist/providers";

// UI locales your want to support
import { en, fr } from "vtec-admin/dist/locales";

// Custom authenticated admin pages as dashboard, profile, etc.
import routes from "@/router/admin";

// Resources to register into VA
import resources from "@/resources";

// Main required Vue libraries instances
import router from "@/router";
import store from "@/store";
import i18n from "@/i18n";

// Axios as default HTTP client
import axios from "axios";

// Load Admin UI components
Vue.use(VtecAdmin);

// Create global axios instance, it will bridged into above providers
const http = axios.create();
Vue.prototype.$axios = http;

// Main VA constructor that will build resources routes and modules
export default new VtecAdmin({
  router,
  store,
  i18n,
  title: "My Admin App",
  routes,
  locales: { en, fr },
  translations: {
    en: i18n.t("locales.english"),
    fr: i18n.t("locales.french"),
  },
  authProvider: sanctumAuthProvider(http),
  dataProvider: laravelDataProvider(http),
  fileBrowserUrl: '/elfinder/tinymce5',
  resources,
});
```

The main steps are :

* Register all third-party components as Portal Vue, Vuedraggable as well as CRUD pages resources from `resources` directory.
* Import custom CSS.
* Load providers, locales, admin routes.
* Load resources you want to register.
* Get current instances of Vue Route, Vuex and Vue I18n.
* Load VA UI components.
* Initiate VA by his constructor.

:::tip BOILERPLATE
All this boring stuf as well as all next pieces of code shown in this page are already prepared for you by the offical [Vue CLI Plugin](https://www.npmjs.com/package/vue-cli-plugin-vtec-admin), go to [Getting Started](getting-started) in order to get in through.
:::

## Components & resources loading

In order to work, you need to load all Vuetify components used by VA manually. Indeed, as VA is precompiled as a standalone library, [a-la-carte](https://vuetifyjs.com/en/customization/a-la-carte/#a-la-carte-treeshaking) (the Vuetify treeshaking system) will not work as-is. All you have to do is to import `vtec-admin/dist/vuetify` which will load all necessary components inside `src/plugins/vuetify.js` file. You can remove `Vue.use(Vuetify)` line as it's already done by the import.

> TL;DR : add `import "vtec-admin/dist/vuetify"` and remove `Vue.use(Vuetify)` inside **`src/plugins/vuetify.js`**

:::warning IMPORT LOCATION
This import line should be in `src/plugins/vuetify` for avoiding CSS overrides issues
:::

Next, you have to import VA loader which import some external third-party components as well as all your CRUD pages, which will avoid us to the immensely boring manual import.

> TL;DR : add `import "vtec-admin/dist/loader"` inside **`src/plugins/vuetify.js`**

Finally, in you entrypoint, don't forget to add `vuetify` and `admin` into main Vue constructor options. It will register `$admin` global object into all of your Vue components, which allows you to use some useful helper functions.

**`src/main.js`**

```js{2-3,10-11}
// ...
import vuetify from "./plugins/vuetify";
import admin from "./plugins/admin";
// ...

new Vue({
  router,
  store,
  i18n,
  vuetify,
  admin,
  render: (h) => h(App),
}).$mount("#app");
```

## Instantiation

In order to operate, VtecAdmin constructor needs all of this parameters :

| Property           | Type         | Description                                                                                                                                                        |
| ------------------ | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **router**         | `VueRouter`  | Vue Router instance, which can contains all your public custom routes                                                                                              |
| **store**          | `Vuex.Store` | Vue Store instance, which can contains all your custom modules, for automatic resource API modules bridge registering                                              |
| **i18n**           | `VueI18n`    | Vue I18n instance, which can contains all your custom localized labels, for full internationalization support. More detail [here](i18n)                            |  |
| **title**          | `string`     | Title of your admin app, will be show on app bar header and document title after page title                                                                        |
| **routes**         | `object`     | List of authenticated routes, which should inherit from an [admin layout](components/layout). All resources routes CRUD pages will be registered here as children. |
| **locales**        | `object`     | At least one provided VA locales, only `en` and `fr` are 100% supported as explained [here](i18n#ui)                                                               |
| **translations**   | `object`     | All supported traductions for your resources. More detail [here](i18n#resources)                                                                                   |
| **authProvider**   | `object`     | [Auth](authentication) provider that must implements [auth contract](authentication#api-contract)                                                                  |
| **dataProvider**   | `object`     | [Data](data-providers) provider that must implements [data contract](data-providers#api-contract)                                                                  |
| **resources**      | `array`      | A resources array which contain all resources to administer. More detail of resource object structure [here](resources)                                            |
| **fileBrowserUrl** | `string`     | Optional file browser URL, which will appear on included TinyMCE file picker                                                                                       |
| **canAction**      | `function`   | Callback for [advanced permissions](authorization#advanced-usage) testing for each action of any resources                                                         |

![instantiation](/diagrams/instantiation.svg)

> Vtec Admin will transform your resources into client-side CRUD routes and valid Vuex modules for data fetching. This modules will be able to seamlessly communicate to your API server thanks to your injected providers which will do the conversion work. See [how it works](./#how-it-works).

### Vue Router

Your main Vue Router should only have public pages (or all other non-vtec-admin related pages). This pages are totally free of any Admin Layout, so you can use your own layout.  
You can even create here you frontend site app here if you really don't care about SEO, although it is not really recommended as the frontend bundle size would include the admin, unless you use the [Vue CLI multi-page feature](https://cli.vuejs.org/config/#pages)...

VA will need full instanciated Vue Router in order to add his builded CRUD resources routes via [`addRoutes`](https://router.vuejs.org/api/#router-addroutes). Here are a basic example :

**`src/router/index.js`**

```js
import Vue from "vue";
import VueRouter from "vue-router";
import Login from "@/views/Login";
import i18n from "@/i18n";

Vue.use(VueRouter);

const routes = [
  {
    path: "/login",
    name: "login",
    component: Login,
    meta: {
      title: i18n.t("routes.login"),
    },
  },
];

export default new VueRouter({
  mode: "history",
  base: process.env.BASE_URL,
  routes,
});
```

:::tip
Generally, you should have least a login page, which also can have any registration or password reset included. [Check here](authentication#login-page) for more info in his integration within your auth provider.
:::

### Vuex

This is here that you can put all of your custom store modules. Your are free to use them anywhere, whether it be on your custom pages or resources pages.

VA will need full instanciated Vuex in order to register his builded API resources modules via [`registerModule`](https://vuex.vuejs.org/api/#registermodule). Here are a basic example :

**`src/store/index.js`**

```js
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

export default new Vuex.Store({
  state: {},
  mutations: {},
  actions: {},
  modules: {},
});
```

### Authenticated routes

VA needs a basic route format object separate from above public routes that will contains all authenticated route as children. This separation is mainly due because of because of [Vue Router children registration limitation](https://github.com/vuejs/vue-router/issues/1156). That's why it must be at least for now injected manually into [VtecAdmin constructor](admin).

This is here you can put all your authenticated custom pages as dashboard, [profile page](authentication#profile-page), etc.

**`src/router/admin.js`**

```js
import AdminLayout from "@/layouts/Admin";
import Profile from "@/views/Profile";
import Dashboard from "@/views/Dashboard";
import i18n from "@/i18n";

export default {
  path: "/",
  name: "home",
  redirect: "/dashboard",
  component: AdminLayout,
  meta: {
    title: i18n.t("routes.home"),
  },
  children: [
    {
      path: "/dashboard",
      name: "dashboard",
      component: Dashboard,
      meta: {
        title: i18n.t("routes.dashboard"),
      },
    },
    {
      path: "/profile",
      name: "profile",
      component: Profile,
      meta: {
        title: i18n.t("routes.profile"),
      },
    },
  ],
};
```

> As you can see here, this is a single route which use a fully customizable `AdminLayout` component which allows all children pages to inherit of all admin authenticated structure, with app bar header, sidebar menu, etc. More information [here](components/layout).

:::tip PAGE TITLE
Use title property inside meta route object for title page. This title will be appear on document title as well as admin breadcrumb.
:::

:::tip DEFAULT REDIRECT
Add an automatic redirect on the main parent route towards authenticated home page, i.e. most of the time your dashboard page.
:::