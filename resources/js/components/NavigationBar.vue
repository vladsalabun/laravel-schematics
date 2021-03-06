<template>
    <nav
        id="header"
        class="fixed w-full z-20 top-0 bg-white border-b border-gray-400"
    >
        <div
            class="w-full flex flex-wrap justify-between mt-0 py-4"
        >
            <div class="pl-4 flex items-center">
                <img class="logo mr-2" src="vendor/schematics/images/icon.png">

                <a class="text-gray-900 no-underline text-lg font-thin">
                    <span class="font-bold">Laravel</span> Schematics
                </a>

                <div class="flex-1 w-full mx-auto max-w-sm content-center">
                    <div class="relative pull-right pl-4 pr-4">
                        <input
                            @keydown.enter="search()"
                            v-model="searchFor"
                            type="search"
                            placeholder="Search..."
                            class="w-full bg-gray-100 text-sm text-gray-800 transition border focus:outline-none focus:border-purple-500 rounded py-1 px-2 pl-10 appearance-none leading-normal"
                        >

                        <div class="absolute search-icon" style="top: 0.375rem; left: 1.75rem;">
                            <svg class="fill-current pointer-events-none text-gray-800 w-4 h-4"
                                 xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20">
                                <path
                                    d="M12.9 14.32a8 8 0 1 1 1.41-1.41l5.35 5.33-1.42 1.42-5.33-5.34zM8 14A6 6 0 1 0 8 2a6 6 0 0 0 0 12z"/>
                            </svg>
                        </div>
                    </div>
                </div>

                <ul class="flex justify-center">
                    <li class="text-center flex-2">
                        <a class="block py-2 pl-4">
                            <span id="model-count">
                                {{ models }}
                            </span> Model{{ models === 1 ? '' : 's' }}
                        </a>
                    </li>

                    <li class="text-center flex-2">
                        <a class="block py-2 pl-2">
                            <span id="migration-count">
                                {{ migrations.created }}
                            </span> Migration{{ migrations.created === 1 ? '' : 's' }}

                            <span v-if="migrations.created !== migrations.run" class="text-black text-xs">
                                (
                                    <button
                                        aria-label="Migrations found as file"
                                        data-balloon-pos="down"
                                        class="tooltip migration-warning"
                                    >
                                    <span id="idle-migrations-count">
                                        {{ migrations.created }}
                                    </span>
                                        <i class="fa fa-file-alt text-gray-400"/>
                                    </button>

                                    <button
                                        aria-label="Migrations found in migrations table"
                                        data-balloon-pos="down"
                                        class="tooltip migration-warning"
                                    >
                                    <span id="redundant-migrations-count">
                                        {{ migrations.run }}
                                    </span>
                                        <i class="fa fa-database text-gray-400"/>
                                    </button>
                                )
                            </span>
                            <span v-else>
                                <i class="fa fa-check text-xs text-green-400"/>
                            </span>
                        </a>
                    </li>
                </ul>
            </div>

            <buttons/>
        </div>
    </nav>
</template>

<script>
    import Buttons from './NavigationBar/Buttons.vue';

    export default {
        name: "nav-bar",

        components: {
            'buttons': Buttons,
        },

        data() {
            return {
                searchFor: localStorage.getItem(`schematics-settings-search-tab-${Schematics.activeTab}`) || '',
                models: Schematics.models.length,
                migrations: Schematics.migrations,
            }
        },

        created() {
            EventBus.$on('refresh-navbar', (data) => {
                this.models = data.models.created;
                this.migrations = data.migrations;
            });
        },

        destroyed() {
            EventBus.$off('refresh-navbar');
        },

        mounted() {
            this.search();
        },

        methods: {
            search() {
                let $models = this.$models().unhidden();

                EventBus.$emit('loading', true);

                localStorage.setItem(`schematics-settings-search-tab-${Schematics.activeTab}`, this.searchFor);

                if (this.searchFor.trim().length) {
                    let search = this.searchFor.toLowerCase();

                    $models.addClass('filtered').hide();

                    try {
                        new RegExp(search);
                    } catch (e) {
                        console.error(`Invalid format: ${search}`);
                        EventBus.$emit('alert', `Invalid search format: ${search}`, 'error');

                        $models.show();
                        EventBus.$emit('plumb');
                        return;
                    }

                    const $found = $models.filter(function () {
                        return $(this).data('model')
                            .toLowerCase()
                            .match(new RegExp(search));
                    });

                    $found.removeClass('filtered').show();
                } else {
                    $models.removeClass('filtered').show();
                }

                this.$models().count().text(this.$models().visible().length);

                EventBus.$emit('plumb');
            }
        },
    }
</script>

<style>
    .logo {
        margin-top: -5px;
        width: 40px;
        opacity: .8;
    }

    #header {
        max-height: 70px;
        min-width: 920px;
        z-index: 1000;
    }

    .icon {
        color: #9F7AEA
    }

    .migration-warning {
        cursor: help !important;
    }

    #model-count {
        color: #9F7AEA;
    }

    #migration-count {
        color: #9F7AEA;
    }

    #idle-migrations-count {
        color: #ea3d4c;
    }

    #redundant-migrations-count {
        color: #ea3d4c;
    }
</style>
