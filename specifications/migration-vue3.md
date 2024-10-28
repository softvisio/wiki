patch - softvisio/vue-ext/resources/vue.config.js - remove vue-loader-16, when v16 will be default;

---

-   main - app init;

-   viewport - move data to created;

-   this.$. -> this.$util.;

-   Vue.extend - removed;

-   settings dialog template - rename Dialog -> SettingsDialog;

-   declare component events with `emit: ["name1", "name2"]`;

-   .$o -> events api removed;

-   .$emit -> need to declare events names with `emit: [...]`;

-   slot attribute was deprecated;

-   $global -> $events;

-   $destroy() -> $unmount();

-   dialogs - remove displayed=";

-   addVue -> $mount;

-   remove fullscreen="true";

-   replce import( -> defineAsyncComponent(;
