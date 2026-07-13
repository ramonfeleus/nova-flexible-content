<template>
    <div class="relative" v-bind:class="top ? ' mb-4' : ''" v-if="layouts">
        <div v-if="isLayoutsDropdownOpen && layouts.length > 1"
                ref="dropdown"
                class="z-20 absolute rounded-lg shadow-lg max-w-full max-h-search overflow-y-auto border border-40"
                v-bind:class="dropdownClasses"
                :style="{ maxHeight: dropdownMaxHeight }"
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
                dropdownMaxHeight: null,
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

                if (!this.isLayoutsDropdownOpen) {
                    this.dropdownOrientation = 'bottom';
                    this.dropdownMaxHeight = null;
                    return;
                }

                this.$nextTick(() => {
                    const buttonRect = this.$refs.dropdownButton.$el.getBoundingClientRect();
                    const spaceBelow = window.innerHeight - buttonRect.bottom;
                    const spaceAbove = buttonRect.top;

                    // Open toward whichever side has more room and cap the height to that
                    // space, so the list never runs off-screen and overflow-y-auto can scroll.
                    // ponytail: 16px = the mt-3/mb-3 gap + a little breathing room.
                    if (spaceBelow >= spaceAbove) {
                        this.dropdownOrientation = 'bottom';
                        this.dropdownMaxHeight = (spaceBelow - 16) + 'px';
                    } else {
                        this.dropdownOrientation = 'top';
                        this.dropdownMaxHeight = (spaceAbove - 16) + 'px';
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
                this.dropdownMaxHeight = null;
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
