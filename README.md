# Vue.js Fundamentals

## Create git repository, README and push to github
```sh
echo "# Vuejs_Fundamentals" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:tanamim/Vuejs_Fundamentals.git
git push -u origin master
```

## How to install Vue cli and start the initial app

`npm install -g @vue/cli`
`npm run serve`

## Creating Vue.js Components and Using Template Syntax

Component name should be two words (HomePage.vue, RobotBuilder.vue)

`v-bind` is :

`v-on` is @

`v-if` `v-show` is conditional display, where `v-show` uses css hidden to improve performance by not using rendering.

You should not use `v-if` and `v-for` at the same time.

`<style scope>` is a local css

`<style>` is a global css

Even if parent scope is used, chiled scope will inherit a parent css property such as font-color where border does not.

To target child elements in style, use deep selector `>>>` or `/deep/` in parent css

Binding styles is `:style="{[background-color]: 'red'}"`

Binding styles is `:style="{backgroundColor: 'red'}"`       // Vue allows to use CamelCase to binding css attributes that have hyphens in them

Binding styles is `:style="{background: '3px solid red'}"`

Binding styles is `:style="headBorderStyle"`                // Computed property

Binding styles is `:style="[headBorderStyle, moreStyles]"`  // Computed property 2 (Second one overwrites the first one)

Conditional style binding using computed property is:
```Javascript
computed: {
    headBorderStyle() {
        return {
            border: this.selectedRobot.head.onSale ? '3px solid red' : '3px solid #aaa',
        }
    };
}
```

Binding class is `:class="{'sale-border': selectedRobot.head.onSale}"` // apply only if onSale

Binding class is `:class="[saleBorderClass]"` // array expression

Binding class is `:class="[saleBorderClass, 'top', 'part']"`           // array expression allows multiple class attibution

Conditional class binding using computed property is:
```Javascript
computed: {
    saleBorderClass() {
        return this.selectedRobot.head.onSale ? 'sale-border' : '';
    }
}
```

[Instance-Lifecycle-Hoocs](https://vuejs.org/v2/guide/instance.html#Instance-Lifecycle-Hooks)
![Lifecycle-Hooks](https://vuejs.org/images/lifecycle.png)

Calling API is done somewhere in the lifecycle.
```Javascript
created() {
    console.log('component created, now API calling')
}
```
## Enabling Inter-component Communication

### How to use child components
```Javascript
<PartSelector />

import PartSelector from './PartSelector.vue';

export default {
  components: { PartSelector },
}
```

### How to Send Data by binding props to child components
```html
<!-- ./build/RobotBuilder.vue -->
<!-- Parent Component Binding -->
<PartSelector
    :parts="availableParts.heads"
    position="top"/>

// Child Component Props and Using property by `this.parts`
<template>
  <div :class="position">
  </div>
</template>

<script>
export default {
  props: ['parts', 'position'],
  methods: {
    selectNextPart() {
      this.selectedPartIndex = getNextValidIndex(
        this.selectedPartIndex,
        this.parts.length,
      );
    },
}
</script>
```

### How to Validate Props - Emit error if position="" (blank string)
```Javascript
// Parent: ./build/RobotBuilder.vue
<PartSelector
    :parts="availableParts.arms"
    position="left"/>

// Child: ./build/PartSelector.vue
  props: {
    position: {
      type: String,
      required: true, 
      validator(value) { // Validation Function
        return ['left', 'right', 'top', 'bottom', 'center'].includes(value);
      },
    },
  },
```

### How to Receive Data from child components via emit
```Javascript
// Parent
<PartSelector
    :parts="availableParts.heads"
    position="top"
    @partSelected="part => selectedRobot.head = part"/>

// Child
methods: {
    selectNextPart() {
      this.$emit('partSelected', this.selectedPart);
    }
}
```

### Life Cycle Hook
```Javascript
  created() {
    this.emitSelectedPart();
  },
  updated() {
    this.emitSelectedPart();
  },
  methods: {
    emitSelectedPart() {
      this.$emit('partSelected', this.selectedPart);
    },
  }
```

### How to Inject Parent Content into a Child Component within <slot> tag
```html
<!-- Parent -->
<template>
    <CollapsibleSection>
    </CollapsibleSection>
    <CollapsibleSection>
        <div>Injecting Content</div>
    </CollapsibleSection>
</template>

<script>
import CollapsibleSection from '../shared/CollapsibleSection.vue';
</script>

<!-- Child has Collapse/Expand toggle and show Injected Content between <slot></slot> -->
<template>
    <div>
        <div class="header">
            <span v-if="open" @click="open = !open" >&#x25B2; Collapse</span>
            <span v-if="!open" @click="open = !open">&#x25BC; Expand</span>
        </div>
        <!-- Togglable -->
        <slot v-if="open">
            <div>Default Content</div>
        </slot>
    </div>
</template>

<script>
export default {
  name: 'CollapsibleSection',
  data() {
    return { open: true };
  },
};
</script>
```

## Routing from Page to Page

`npm install vue-router --save`

### Adding Routing to Your App
```Javascript
// index.js
import Vue from 'vue';
import Router from 'vue-router';

import HomePage from '../home/HomePage.vue';

Vue.use(Router);

export default new Router({
  routes: [{
    path: '/',
    name: 'Home',
    component: HomePage,
  }],
});

// main.js
import Vue from 'vue';
import App from './App.vue';

import router from './router';

Vue.config.productionTip = false;

new Vue({
  render: h => h(App),
  router,
}).$mount('#app');
```

Inject `<router-view>` into template so router can change the contents

```html
<!-- App.vue -->
<template>
  <div id="app">
    <main>
      <!-- <RobotBuilder /> -->
      <router-view/>
    </main>
  </div>
</template>
```

### How to create a link to vue routes
(use `:to` to bind an object. if bind a string, use only `to`)
```html
<!-- HomePage.vue -->
<template>
  <div class="home">
    <div class = "get-started">
      <!-- <a href="/#/build">Get started</a> building your first robot! -->
      <router-link to="/build">Get started</router-link> building your first robot!
    </div>
  </div>
</template>


<!-- App.vue -->
<template>
  <div id="app">
    <header>
      <nav>
        <ul>
          <li class="nav-item">
            <!-- <router-link to="/"> -->
            <router-link :to="{name: 'Home'}">
              <img class="logo" src="./assets/build-a-bot-logo.png"/>
              Build-a-Bot
            </router-link>
          </li>
        </ul>
      </nav>
    </header>
    <main>
    <!-- <HomePage msg="Welcome to My HomePage App"/> -->
      <!-- <RobotBuilder /> -->
      <router-view/>
    </main>
  </div>
</template>
```

### Styling Links Based on the Active Route
(use Vue special css `.router-link-active`) and use `exact` keyword in router-link
```html
<!-- App.vue -->
<router-link class="nav-link" :to="{name: 'Build'}" exact>
```


### Navigating from Code
(use `$router.push` to call router function to change content)
```html
<!-- Partselector.vue -->
  <!-- <img @click="$router.push('/parts')" :src="selectedPart.src" title="arm"/> -->
  <img @click="showPartInfo()" :src="selectedPart.src" title="arm"/>

  methods: {
    showPartInfo() {
      this.$router.push('/parts');
    },
  }
```

### Working with Route Params
(Send parameter like this: `/parts/heads/1` and receive with `this$route.params`)
```Javascript
// PartSelector.vue
  methods: {
    showPartInfo() {
      // this.$router.push('/parts');
      this.$router.push({
        name: 'Parts',
        params: {
          id: this.selectedPart.id,
          partType: this.selectedPart.type,
        },
      });
    },
  }
```

```html
<!-- PatrInfo.vue -->
<template>
    <div>
        <h1>{{part.title}}</h1>
        <div>
            {{part.description}}
        </div>
    </div>
</template>

<script>
import parts from '../data/parts';
export default {
  name: 'PartInfo',
  computed: {
    part() {
    //   const partType = this.$route.params.partType;
    //   const id = this.$route.params.id;
      const { partType, id } = this.$route.params;
      return parts[partType].find(part => part.id === +id); // `+` cast string to int
    },
  },
};
</script>
```

### Navigating from Code & Working with Route Params using `router-link`
(Allow to do without `@click` binding. Note that `:to` binding doesn't work with computed property so the string should be expressed as object.)
```html
<!-- PartSelector.vue -->
<template>
    <router-link :to="{
      name: 'Parts',
      params: {
        id: this.selectedPart.id,
        partType: this.selectedPart.type,
      }
    }">
    <img :src="selectedPart.src" title="arm"/>
    </router-link>
</template>
```

### Passing Params as Props
```Javascript
// index.js
export default new Router({
  routes: [{
    path: '/',
    name: 'Home',
    component: HomePage,
  }, {
    path: '/build',
    name: 'Build',
    component: RobotBuilder,
  }, {
    path: '/parts/:partType/:id',
    name: 'Parts',
    component: PartInfo,
    props: true,
  }],
});

// PartInfo.vue
export default {
  name: 'PartInfo',
  // props: ['partType', 'id'], // declare props
  props: { // props can add validator for parameters
    partType: { type: String },
    id: {
      type: [Number, String],
      validator(value) {
        return Number.isInteger(Number(value));
      },
    },
  },
  computed: {
    part() {
    //   const { partType, id } = this.$route.params;
      const { partType, id } = this; // receive params as props object
      return parts[partType].find(part => part.id === +id); // cast string to int
    },
  },
};
```

### Using Nested Routes
(Add BrowsePart route and `<router-view></router-view>` for nesting children content, then insert `<router-link></router-link> for router to link each content`)
```html
<template>
  <div>
    <h1>Browse Parts</h1>
    <ul class="menu">
      <!-- <li>Heads</li> -->
      <!-- <li>Arms</li> -->
      <!-- <li>Torsos</li> -->
      <!-- <li>Bases</li> -->
      <li><router-link :to="{name: 'BrowseHeads'}">Heads</router-link></li>
      <li><router-link :to="{name: 'BrowseArms'}">Arms</router-link></li>
      <li><router-link :to="{name: 'BrowseTorsos'}">Torsos</router-link></li>
      <li><router-link :to="{name: 'BrowseBases'}">Bases</router-link></li>
    </ul>
    <router-view></router-view>
  </div>
</template>
```

```Javascript
// ./router/index.js
import BrowseParts from '../parts/BrowseParts.vue';
import RobotHeads from '../parts/RobotHeads.vue';
import RobotArms from '../parts/RobotArms.vue';
import RobotTorsos from '../parts/RobotTorsos.vue';
import RobotBases from '../parts/RobotBases.vue';

export default new Router({
  routes: [{
    path: '/',
    name: 'Home',
    component: HomePage,
  }, {
    path: '/build',
    name: 'Build',
    component: RobotBuilder,
  }, {
    path: '/parts/browse',
    name: 'BrowseParts',
    component: BrowseParts,
    children: [
      {
        name: 'BrowseHeads',
        path: 'heads',
        component: RobotHeads,
      }, {
        name: 'BrowseArms',
        path: 'arms',
        component: RobotArms,
      }, {
        name: 'BrowseTorsos',
        path: 'torsos',
        component: RobotTorsos,
      }, {
        name: 'BrowseBases',
        path: 'bases',
        component: RobotBases,
      },
  }, {
    path: '/parts/:partType/:id',
    name: 'Parts',
    component: PartInfo,
    props: true,
  }],
});
```

### Using Named Views
(Not a child-parent relationship. Allowing multiple `components` for one path with rowter-view name)

```html
<!-- APP.vue -->
<template>
  <div id="app">
    <div class="container">
      <aside class="aside">
        <router-view name="sidebar"/>
      </aside>
      <main>
        <router-view/>  <!-- Hidden: name="default" -->
      </main>
    </div>
  </div>
</template>
```

```Javascript
// index.js
import SidebarStandard from '../sidebars/SidebarStandard.vue';
import SidebarBuild from '../sidebars/SidebarBuild.vue';

Vue.use(Router);

export default new Router({
  routes: [{
    path: '/',
    name: 'Home',
    // component: HomePage,
    components: { // `s` plural, not singular
      default: HomePage,
      sidebar: SidebarStandard,
    },
  }, {
    path: '/build',
    name: 'Build',
    // component: RobotBuilder,
    components: {
      default: RobotBuilder,
      sidebar: SidebarBuild,
    },
  },
}
```

### Enabling HTML5 History Mode
(Remove `#` from URL, but Server Config is needed to hendle 404 Error.)
```Javascript
// index.js
Vue.use(Router);

export default new Router({
  mode: 'history',
  routes: [...]
})
```
[See Docs Here](https://router.vuejs.org/guide/essentials/history-mode.html#example-server-configurations)


### Preventing Pages from Loading
(beforeEnter guard: Just block routing if invalid)
```Javascript
// ./router/index.js
export default new Router({
  routes: [{
    path: '/parts/:partType/:id',
    name: 'Parts',
    component: PartInfo,
    props: true,
    beforeEnter(to, from, next) {
      const isValidId = Number.isInteger(Number(to.params.id));
      next(isValidId); // go to next if true, which is id is Integer
    },
  }],
});
```

### Preventing Navigation Away
(beforeLeave guard: Show message box to confirm)
```Javascript
// ./builder/RobotBuilder.vue
export default {
  name: 'RobotBuilder',
  beforeRouteLeave(to, from, next) {
    // if (this.addedToCart) {
    if (this.cart.length > 0) { // I think no use of addedToCart is ok
      next(true);
    } else {
      /* eslint no-alert: 0 */
      /* eslint no-restricted-globals: 0 */
      const response = confirm('You have not added your robot to your cart, are yo usure you want to leave?');
      next(response);
    }
  },
}
```

## Managing State and Server Communcation with Vuex

1. Installing Vuex
`npm install vuex --save`
`npm install vuex@3.0.1 --save`

2. Create `store` directory and `index.js` for exporting `Vuex.Store`
```Javascript
// ./store/index.js
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export devault new Vuex.Store({

});
```

3. Configure our Vue instance in `main.js` to use our new store
```Javascript
// main.js
import Vue from 'vue';
import App from './App.vue';

import router from './router';
import store from './store';

Vue.config.productionTip = false;

new Vue({
  render: h => h(App),
  router,
  store,
}).$mount('#app');
```

4. Create a state object
```Javascript
// ./store/index.js
export default new Vuex.Store({
  state: {
    cart: [], // default value is needed to detect changes
  },
});
```

5. Changing Vuex Store State with mutations
```Javascript
// ./store/index.js
export default new Vuex.Store({
  state: {
    cart: [], // default value is needed to detect changes
  },
  mutations: {
    addRobotToCart(state, robot) { // 1st arg is state
      state.cart.push(robot);
    },
  },
});

// ./build/RobotBuilder.vue
  methods: {
    addToCart() {
      const robot = this.selectedRobot;
      const cost =
        robot.head.cost +
        robot.leftArm.cost +
        robot.torso.cost +
        robot.rightArm.cost +
        robot.base.cost;
      this.$store.commit('addRobotToCart', Object.assign({}, robot, { cost }));
      // this.cart.push(Object.assign({}, robot, { cost }));
      this.addedToCart = true; // used fore beforeRouteLeave check
    },
  },
```

6. Retrieve data from the store
```html
<!-- ./cart/ShoppingCart.vue -->
<template>
        <tr v-for="(robot, index) in cart" :key="index"> // cart is computed property
          <td class="robot-title">
            {{robot.head.title}}
          </td>
          <td class="cost">
            {{robot.cost}}
          </td>
        </tr>
</template>

<script>
export default {
  name: 'Cart',
  computed: {
    cart() {
      return this.$store.state.cart; // reading from $store object
    },
  },
};
</script>
```

Creating a route to shopping cart component
```Javascript
// ./router/index.js
import ShoppingCart from '../cart/ShoppingCart.vue';

export default new Router({
  routes: [{
    path: '/cart',
    name: 'Cart',
    component: ShoppingCart,
  }],
})
```

### Using Vuex Getters to Return Calculated data
(to avoid complexity such as `return this.$store.state.cart`)
```Javascript
// ./store/index.js
export default new Vuex.Store({
  state: {
    cart: [], // default value is needed to detect changes
  },
  mutations: {
    addRobotToCart(state, robot) { // 1st arg is state
      state.cart.push(robot);
    },
  },
  getters: {
    cartSaleItems(state) {
      return state.cart.filter(item => item.head.onSale);
    },
  },
});
```
```html
<!-- ./cart/ShoppingCart.vue -->
<template>
  <tbody>
    <tr v-for="(robot, index) in cartSaleItems" :key="index">  // computed property
      <td class="robot-title">
        {{robot.head.title}}
      </td>
      <td class="cost">
        {{robot.cost}}
      </td>
    </tr>
  </tbody>
</template>

<script>
export default {
  name: 'Cart',
  computed: {
    cart() {
      return this.$store.state.cart;
    },
    cartSaleItems() {
      // return this$store.cart.filter(item => item.head.onSale);
      return this.$store.getters.cartSaleItems; // calling getter
    },
  },
};
</script>
```

### Using Actions to work with Asynchronous API calls

1. Set up build-a-bot-server
`git clone git@github.com:jmcooper/build-a-bot-server.git`
`cd build-a-bot-server`
`npm i`
`npm start` => Server listening on port 8081!
`localhost:8081/api/parts` on browser to get a JSON

1. Set up Axios to make HTTP calls
`npm install axios --save`

In development, we use different port to run App server (8080) and API server (8081). It has a CORS issue.

Vue provide a way to configure a built-in proxy so that we can proxy the API through our dev server that our Vue app is being served from.

This will solve our cors issue locally and enable us to just use a relative URL.

To do that, we need to create a file at the root of our project named vue.config.js.

```Javascript
// ./store/index.js
import axios from 'axios';

export default new Vuex.Store({
  actions: {
    getParts({ commit }) {
      // axios.get('http://localhost:8081/api/parts');  // port is different so it has CORS issues
      axios.get('/api/parts'); // Vue has a build-in proxy
    },
  },
});

// ../vue.config.js (This is webpack.config. Vue is using the webpack-dev-server.)
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: 'http://localhost:8081',
        changeOrigin: true,
      },
    },
  },
};
```

2. Create State Updating Mutation
```Javascript
// ./store/index.js
export default new Vuex.Store({
  state: {
    parts: null, // need to set devault value to track commit
  },
  mutations: {
    updateParts(state, parts) { // Usually, 1st arg `state` should never be reassigned.
      state.parts = parts; // but we need to reassign here. So configure in eslintrc.js file. `'no-param-reassign': 0`
    },
  },
});

// ../eslintrc.js
module.exports = {
  rules: {
    'linebreak-style': 0,   // added
    'no-param-reassign': 0, // added to allow `state` reassign
  },
}
```

3. Create a Lifecycle hook to kick off API calls when RobotBuilder is created
```Javascript
// ./build/RobotBuilder.vue
export default {
  created() {
    this.$store.dispatch('getParts');
  },
}
```

4. After finishing axios API call, commit to our store through mutation
```Javascript
// ./store/index.js
export default new Vuex.Store({
  state: {
    parts: null, // need to set devault value to track commit
  },
  mutations: {
    updateParts(state, parts) { // Usually, 1st arg `state` should never be reassigned.
      state.parts = parts; // but we need to reassign here. So configure in eslintrc.js file. `'no-param-reassign': 0`
    },
  },
  actions: {
    getParts({ commit }) {
      // axios.get('http://localhost:8081/api/parts');  // port is different so it has CORS issues
      axios.get('/api/parts') // Vue has a build-in proxy
        .then(result => commit('updateParts', result.data))
        .catch(console.error);
    },
  },
});
```

5. Make a computed property to use data from API calls
```html
 <!-- ./build/RobotBuilder.vue -->
<template>
    <div class="top-row">
        <PartSelector
            :parts="availableParts.heads"
            position="top"
            @partSelected="part => selectedRobot.head = part"/>
    </div>
</template>
```
```Javascript
// ./build/RobotBuilder.vue
computed: {
    availableParts() {
      return this.$store.state.parts;
    },
  }
```

6. Add conditional rendering to avoid error accessing default null state
```html
<template>
  <!-- don't render until availableParts gets value from asynchronous call  -->
  <div v-if="availableParts" class="content">
    <div class="top-row">
        <PartSelector
            :parts="availableParts.heads"
            position="top"
            @partSelected="part => selectedRobot.head = part"/>
    </div>
```

7. Replace Hard-coded objects with computed property, using mixins to help many replacements
```Javascript
// ./parts/get-parts-mixin.js
export default {
  created() {
    this.$store.dispatch('getParts');
  },
  computed: {
    parts() {
      // to avoid null data access error, give a reasonable default value before async call returns
      return this.$store.state.parts || {
        heads: [],
        arms: [],
        torsos: [],
        bases: [],
      };
    },
  },
};

// ./parts/PartInfo.vue
import getPartsMixin from './get-parts-mixin';
export default {
  name: 'PartInfo',
  mixins: [getPartsMixin],
  computed: {
    part() {
      // return parts[partType].find(part => part.id === +id); // cast string to int
      // we imported mixins, so `parts` can be used in <template>, but in <script>
      // we need `this.parts` because `parts` is not passed as artument
      return this.parts[partType].find(part => part.id === +id);
    },
  },
};
</script>
```

```html
<!-- ./RobotHeads.vue -->
<template>
  <div>
    <!-- `parts` is computed property in mixns, so use `parts.heads` -->
    <!-- <div v-for="(head, idx) in heads" :key="idx"> -->
    <div v-for="(head, idx) in parts.heads" :key="idx">
      <h4>{{head.title}}</h4>
      <div>{{head.description}}</div>
    </div>
  </div>
</template>

<script>
import getPartsMixin from './get-parts-mixin';
export default {
  mixins: [getPartsMixin],
};
</script>

```

### Using Actions to Save Data
1. Create action for axios post, and call commit to update local cart.
```Javascript
// ./store/index.js
  actions: {
    addRobotToCart({ commit, state }, robot) {
      const cart = [...state.cart, robot]; // append robot to cart and reassign
      axios.post('/api/cart', cart)
        .then(() => commit('addRobotToCart', robot));  // update local
    },
  },

// ./build/RobotBuilder.vue
  methods: {
    addToCart() {
      const robot = this.selectedRobot;
      const cost =
        robot.head.cost +
        robot.leftArm.cost +
        robot.torso.cost +
        robot.rightArm.cost +
        robot.base.cost;
      // this.cart.push(Object.assign({}, robot, { cost }));
      // this.$store.commit('addRobotToCart', Object.assign({}, robot, { cost })); // local update
      this.$store.dispatch('addRobotToCart', Object.assign({}, robot, { cost })); // remote api call, then action will commit update.
      this.addedToCart = true; // used fore beforeRouteLeave check
    },
  },

```

2. Redirect after post is succeed by using Promises from Actions
``` Javascript
// ./store/index.js
  actions: {
    addRobotToCart({ commit, state }, robot) {
      const cart = [...state.cart, robot]; // append robot to cart and reassign
      //   axios.post('/api/cart', cart)
      //     .then(() => commit('addRobotToCart', robot)); // update local
      // .then(() => commit('addRobotToCart', robot)); // update local

      /* axios returns Promise, so returning this to dispatcher will tell the Vue object that the action is completed, and then allow redirect */
      return axios.post('/api/cart', cart)
        .then(() => commit('addRobotToCart', robot));
    },
  },
}

// ./build/RobotBuilder.vue
  methods: {
    addToCart() {
      const robot = ...;
      const cost = .../
      this.$store.dispatch('addRobotToCart', Object.assign({}, robot, { cost }))
        .then(() => this.$router.push('/cart')); // redirect after success
    },
  },
```

### Organizing the Store with Modules
(Break a single store into multiple modules for maintainability)

1. Create a module folder inside `store` folder and create a new file for our module
2. Import module to `store/index.js`
```Javascript
// ./store/index.js
import Vue from 'vue';
import Vuex from 'vuex';
import robotsModule from './modules/robots';

Vue.use(Vuex);
export default new Vuex.Store({
  modules: {
    robots: robotsModule,
  },
});

// ./store/modules/robots.js
import axios from 'axios';

export default ({
  state: {
    cart: [], // default value is needed to detect changes
    parts: null, // need to set devault value to track commit
  },
  mutations: {
  ...
  },
  actions: {
  ...
  },
  getters: {
  ...
  },
});
```

### Namespacing Modules
Both users.js and robots.js has actions and its name can be the same, and Vue will execute both when dispaching the action name. In order to differenciate them, we can use Namespaced Modules.

* State is always namespaced even if our module is not set to namespaced.
* For example, namespaced state can be accessed via dot notation: `this.$store.state.robots.cart`.
* Mutations, Actions, Getters are only namespaced if you set `namespaced: true`.
* For example, namespaced getters are accessed by string reference: `this.$store.getters['robots/cartSaleItems']`.

```Javascript
// ./store/modules/robots.js
export default ({
  namespaced: true,
});

// ./build/RobotBuilder.vue
export default {
  name: 'RobotBuilder',
  created() {
    // this.$store.dispatch('getParts');
    this.$store.dispatch('robots/getParts'); // using namespaced module
  },
})

// ./cart/ShoppingCart.vue
export default {
  name: 'Cart',
  computed: {
    cart() {
      return this.$store.state.robots.cart; // namespaced state is dot-notation
    },
    cartSaleItems() {
      // return this.$store.getters.cartSaleItems; // calling getter
      return this.$store.getters['robots/cartSaleItems']; // namespaced getter
    },
  },
};
```

* Calling getter of rootState (an example is root/users are both in the non-name space)
```Javascript
// ./store/modules/users.js
  getters: {
    // foo(state) { // local state
    foo(state, getters, rootState) { 
      return `users-getter/${rootState.foo}`; // accessing root state property
    },
  },
```

### Using the Vuex MapState Helper
```Javascript
// ./App.vue
import { mapState } from 'vuex';

export default {
  name: 'app',
  computed: {
    ...mapState({
      rootFoo: 'foo',
      usersFoo: state => state.users.foo,
    }), // shorthand version of below
    // rootFoo() {
    //   return this.$store.state.foo;
    // },
    // usersFoo() {
    //   return this.$store.state.users.foo;
    // },
    ...mapState('robots', { robotsFoo: 'foo' }), // only for namespaced state-name mapping
    // robotsFoo() {
    //   return this.$store.state.robots.foo;
    // },
  }
}
```

### Using the Vuex MapGetter Syntax
```Javascript
    ...mapGetters({ rootGetterFoo: 'foo' }),
    ...mapGetters('robots', { robotsGetterFoo: 'foo' }), // namespaced syntax
    // rootGetterFoo() {
    //   return this.$store.getters.foo;
    // },
    // robotsGetterFoo() {
    //   return this.$store.getters['robots/foo'];
    // },
```

### Using the Vuex MapActions Helper (Slightly different syntax)
* Calling dispatch is taken care of for us by the mapActions helper.

```Javascript
// ./build/RobotBuilder.vue
import { mapActions } from 'vuex';

export default {
  name: 'RobotBuilder',
  created() {
    // this.$store.dispatch('robots/getParts'); // using namespaced module
    this.getParts();
  },
  methods: {
    ...mapActions('robots', ['getParts', 'addRobotToCart']),
    addToCart() {
      const robot = this.selectedRobot;
      const cost =
        robot.head.cost +
        robot.leftArm.cost +
        robot.torso.cost +
        robot.rightArm.cost +
        robot.base.cost;
      // this.$store.dispatch('robots/addRobotToCart', Object.assign({}, robot, { cost }))
        // .then(() => this.$router.push('/cart'));
      this.addRobotToCart(Object.assign({}, robot, { cost })) // simplified.
        .then(() => this.$router.push('/cart')); // redirect after success
```

## Creating Custom Directies and Filters

1. Create directive binding file to import
2. Import and use the custom directive (Local directive)
3. Set `directives`
4. Custome directives are not automatically updated when components are updated, so typically we use `bind` and `update` together.
5. Because bind+update is quite popular, we can use shorthand syntax to express for this. Instead of exporting object, in shorthand, we export function.
   
```Javascript
// ./shared/pin-directive.js
export default {
  bind: (element) => {
    element.style.position = 'absolute';
    element.style.bottom = '5px';
    element.style.right = '5px';
  },
};

// ./build/PartSelector.vue
<template>
  <span v-pin class="sale" v-show="selectedPart.onSale">Sale!</span> <!-- `v-pin` added -->
</template>

<script>
import pinDirective from '../shared/pin-directive';

export default {
  directives: { pin: pinDirective },
}
</script>
```

```Javascript
// ./shared/pin-directive.js

// export default {
// ver1 (Descriptive but not flexible)
//     bind: (element) => {
//     element.style.position = 'absolute';
//     element.style.bottom = '5px';
//     element.style.right = '5px';
//   },
// ver2 (Flexible but not automatically updated on update components)
//   bind: (element, binding) => {
//     Object.keys(binding.value).forEach((position) => {
//       element.style[position] = binding.value[position];
//     });
//     element.style.position = 'absolute';
//   },
// ver3 (Works fine but verbose)
//   bind: (element, binding) => {
//     Object.keys(binding.value).forEach((position) => {
//       element.style[position] = binding.value[position];
//     });
//     element.style.position = 'absolute';
//   },
//   update: (element, binding) => {
//     Object.keys(binding.value).forEach((position) => {
//       element.style[position] = binding.value[position];
//     });
//     element.style.position = 'absolute';
//   },
// };

// ver4 (Shorthand of ver3 that is bind+update)
export default function (element, binding) {
  Object.keys(binding.value).forEach((position) => {
    element.style[position] = binding.value[position];
  });
  element.style.position = 'absolute';
}
```

### Making Global Directives
Import and declare directives parameter in `main.js`, not in local vue files.
* Note that local directives use `directives` (plural), and global directives use `directive` (singular)

```Javascript
// ./main.js
import pinDirective from './shared/pin-directive';
Vue.directive('pin', pinDirective);
```

### Creating a Custom Filter
```html
// ./shared/currency-filter.js

<!-- 
// No-arg filter. `amoun` will be provided by pipe `{{ robot.cost | currency }}`
// export default function (amount) {
//   return `$${amount.toFixed(2)}`;
// }

// 1st arg comes from pipe
// 2nd and later args come from the filter arguments `{{ robot.cost | currency('$') }}` 
-->
export default function (amount, symbol) {
  return `${symbol}${amount.toFixed(2)}`;
}

<!-- ./cart/ShoppingCart.vue -->
<template>
  {{ robot.cost | currency('$') }}
</template>

<script>
import currencyFilter from '../shared/currency-filter';

export default {
  name: 'Cart',
  filters: {
    currency: currencyFilter, // Local Filter
  },
}
</script>
```

### Making Global Filters
Import and declare filters parameter in `main.js`, not in local vue files.
* Note that local directives use `filters` (plural), and global directives use `filter` (singular)

```Javascript
// ./main.js
import pinDirective from './shared/currency-filter';
Vue.filter('currency', currencyFilter);
```

## Deploying Vue Applications to Production

### Creating a Basic Vue.js Build using the CLI
`npm run build` to create `./dist/` folder
Any file in ./public/ folder will be included in the ./dist/ folder

### Using environment vars and build modes
For production:
`npm run build` is equal to `npm run build -- --mode=production`

For development:
`npm run serve` creates development build by `npm run build -- --mode=development`

*Important* Setting mode also sets the NODE_ENV variable. If we want to test staging environment prior to production deployment, we can do that with enfironment vairables.

For staging:
`npm run build -- --mode=staging` creates `./dist/` folder but inside has app.js, much like development mode. We can change that to run in production mode.

1. Create `.env.staging` file in the root directory `build-a-bot`
```sh
# ./.env.staging
# This allows staging mode to run in production mode
# file name shoud corresponds to the mode name: `staging`. You can choose any mode name.
NODE_ENV=production
```
2. Run `npm run build -- --mode=staging` to create production mode `dist` folder

3. You can put other environemt variables such as connection information to databases.

4. If you have secret information in environment, don't ever commit without encryption.

5. You can create `.env.development` and set environment variable `VUE_APP_TEST=foo`, and can use it.

*Important* For security reasons, Vue app can only access environment variables with `VUE_APP_` variables, and `NODE_ENV` variable. We usually refer to `VUE_APP_` variables.
Those are useful for server-specific variables to your application.

```Javascript
// ./home/HomePage.vue
<script>
export default {
  name: 'HomePage',
  created() {
    console.log(process.env.VUE_APP_TEST);  // print `foo`
  },
};
</script>
```

### Deploying to a production web server
1. Copy `dist` folder to the web server `cp -r ../build-a-bot/dist .`
2. Add router to the web server in `./index.js`
```Javascript
// ../build-a-bot-server/index.js
// Added to serve production app
app.use('/', express.static('dist', {index: 'index.html'}));
```
3. `npm start` to run our web server

### Handling deep linking
We use history mode in `./router/index.js` to avoid '#' in the URL. But it doesn't allow direct link to the deep url such as `localhost:8081/build`. We have to handle this in our web server. In Expressjs, we have a middleware to handle it.

* Install middleware for the web server by `npm install --save connect-history-api-fallback`

```Javascript
const history = require('connect-history-api-fallback');

// handling deep-linking problem of history mode
app.use(history({ index: '/index.html' })); 

// Added to serve production app
app.use('/', express.static('dist', {index: 'index.html'}));
```

### Inspecting the build-in webpack config
We'd like to be able to edit our webpack config. Vue CLI provides inspect command.
`vue inspect --mode=production > webpack.config.js`

### Customizing the webpack config
Similar to edit and configure webpack config file.

#### Adding a new setting
1. Add a section to `./vue.config.js` file
```Javascript
// ./vue.config.js
module.exports = {
  // Ver1 (Just adding)
  configureWebpack: {
    module: {
      rules: [
        {
          test: /\.coffee$/,
          use: ['coffee-loader'],
        },
      ],
    },
  },
};
```
2. Run `vue inspect --mode=production > webpack.config.js` to Update to see rules are added
3. `Cmd + K Cmd + 0` will collapse all so you can find yours at the end of `module.rules`.

#### Editing an existing setting
Just adding to `./vue.config.js` doesn't work for editing. See the trick below.
1. Store new rule as const.
2. Find the index of the rule that we went to mutate.
3. Replace old rule with the new rule by `splice`

```Javascript
// ./vue.config.js
module.exports = {
  // Ver2 (Add and Edit settings)
  configureWebpack: (config) => { // Just adding
    config.module.rules.push({
      test: /\.coffee$/,
      use: ['coffee-loader'],
    });

    const newRule = { // Editing an existing setting
      test: /\.(png|jpe?g|gif|webp)(\?.*)?$/,
      use: [
        {
          loader: 'url-loader',
          options: {
            limit: 2048, // Modify Here from 4096
            name: 'img/[name].[hash:8].[ext]',
          },
        },
      ],
    };
    
    // find index of the rule that we want to mutate
    const imagesRuleIndex = config.module.rules
      .findIndex(x => x.test.source.includes('png|jpe?g|gif'));

    // find and replace that with new rule
    config.module.rules.splice(imagesRuleIndex, 1, newRule);
  },
};
```
