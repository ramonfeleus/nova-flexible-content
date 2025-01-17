<template>
    <div class="relative" v-bind:class="top ? ' mb-4' : ''" v-if="layouts">
        <div v-if="isLayoutsDropdownOpen && layouts.length > 1"
                ref="dropdown"
                class="z-20 absolute rounded-lg shadow-lg max-w-full max-h-search overflow-y-auto border border-40"
                v-bind:class="dropdownClasses"
        >
            <ul class="list-reset">
                <li v-for="layout in filteredLayouts" class="border-b border-gray-100 dark:border-gray-700" :key="'add-'+layout.name">
                    <a
                        :dusk="'add-' + layout.name"
                        @click="addGroup(layout)"
                        class="cursor-pointer flex items-center hover:bg-gray-50 dark:hover:bg-gray-700 block py-2 px-3 no-underline font-normal bg-white dark:bg-gray-800 z-10">
                        <div><p class="text-90">{{ layout.title }}</p></div>
                    </a>
                </li>
            </ul>
        </div>
        <default-button
            dusk="toggle-layouts-dropdown-or-add-default"
            type="button"
            tabindex="0"
            ref="dropdownButton"
            @click="toggleLayoutsDropdownOrAddDefault"
            v-if="isBelowLayoutLimits && !hideAdd"
        >
            <span>{{ field.button }}</span>
        </default-button>
        <outline-button type="button" @click="clickCollapseAll" style="margin-left: 16px;">{{ field.buttonCollapse }}</outline-button>
        <outline-button type="button" @click="clickExpandAll" style="margin-left: 16px;">{{ field.buttonExpand }}</outline-button>
    </div>
</template>

<script>

    export default {
        props: ['layouts', 'field', 'resourceName', 'resourceId', 'resource', 'errors', 'limitCounter', 'limitPerLayoutCounter', 'top'],

        emits: ['addGroup', 'collapseAll', 'expandAll'],

        data() {
            return {
                isLayoutsDropdownOpen: false,
                dropdownOrientation: 'bottom',
            };
        },

        computed: {
            filteredLayouts() {
                return this.layouts.filter(layout => {
                    const count = this.limitPerLayoutCounter[layout.name];

                    return count === null || count > 0 || typeof count === 'undefined';
                });
            },

            isBelowLayoutLimits() {
                return (this.limitCounter > 0 || this.limitCounter === null) && this.filteredLayouts.length > 0;
            },

            dropdownClasses() {
                return {
                    'mt-3': this.dropdownOrientation === 'bottom',
                    'pin-b': this.dropdownOrientation === 'bottom',
                    'mb-3': this.dropdownOrientation === 'top',
                    'pin-t': this.dropdownOrientation === 'top',
                };
            }
        },

        methods: {
            /**
             * Display or hide the layouts choice dropdown if there are multiple layouts
             * or directly add the only available layout.
             */
            toggleLayoutsDropdownOrAddDefault(event) {
                if (this.layouts.length === 1) {
                    return this.addGroup(this.layouts[0]);
                }

                this.isLayoutsDropdownOpen = !this.isLayoutsDropdownOpen;

                this.$nextTick(() => {
                    if (this.isLayoutsDropdownOpen) {
                        const { bottom: dropdownBottom } = this.$refs.dropdown.getBoundingClientRect();

                        // If the dropdown is popping out of the bottom of the window, pin it to the top of the button.
                        if (dropdownBottom > window.innerHeight) {
                            this.dropdownOrientation = 'top';
                        }
                    } else {
                        // Reset the orientation.
                        this.dropdownOrientation = 'bottom';
                    }
                });
            },

            /**
             * Append the given layout to flexible content's list
             */
            addGroup(layout) {
                if (!layout) return;

                this.$emit('addGroup', layout);
                Nova.$emit('nova-flexible-content-add-group', layout);

                this.isLayoutsDropdownOpen = false;

                // Reset the orientation.
                this.dropdownOrientation = 'top';
            },

          clickCollapseAll() {
            this.$emit('collapseAll');
          },
          clickExpandAll() {
            this.$emit('expandAll');
          }
        }
    }
</script>


<style>
    .top-full {
        top: 100%
    }

    .pin-b {
        top: 100%;
        bottom: auto;
    }

    .pin-t {
        top: auto;
        bottom: 100%;
    }
</style>
