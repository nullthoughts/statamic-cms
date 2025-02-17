<template>
    <div class="h-full">
        <div v-if="initializing" class="loading">
            <loading-graphic />
        </div>

        <data-list
            v-if="!initializing && items.length"
            :rows="items"
            :columns="cols"
            :sort="false"
            :sort-column="sortColumn"
            :sort-direction="sortDirection"
            class="h-full flex flex-col justify-between"
        >
            <div slot-scope="{ }">
                <data-list-table :loading="loading">
                    <template slot="cell-title" slot-scope="{ row: entry }">
                        <div class="flex items-center">
                            <span class="little-dot rtl:ml-2 ltr:mr-2" v-tooltip="getStatusLabel(entry)" :class="getStatusClass(entry)" v-if="! columnShowing('status')" />
                            <a :href="entry.edit_url">{{ entry.title }}</a>
                        </div>
                    </template>
                    <template slot="cell-status" slot-scope="{ row: entry }">
                        <div class="status-index-field select-none" v-tooltip="getStatusTooltip(entry)" :class="`status-${entry.status}`" v-text="getStatusLabel(entry)" />
                    </template>
                </data-list-table>
                <data-list-pagination
                    v-if="meta.last_page != 1"
                    class="py-2 border-t bg-gray-200 rounded-b-lg text-sm dark:bg-dark-650 dark:border-gray-900"
                    :resource-meta="meta"
                    @page-selected="selectPage"
                    :scroll-to-top="false"
                    :show-page-links="false"
                />
            </div>
        </data-list>

        <p v-else-if="!initializing && !items.length" class="p-4 pt-2 text-sm text-gray-600">
            {{ __('There are no entries in this collection') }}
        </p>

    </div>
</template>

<script>
import Listing from '../Listing.vue';

export default {

    mixins: [Listing],

    props: {
        collection: String,
        additionalColumns: Array,
    },

    data() {
        return {
            cols: [{ label: "Title", field: "title", visible: true }, ...this.additionalColumns],
            listingKey: 'entries',
            requestUrl: cp_url(`collections/${this.collection}/entries`),
        }
    },

    methods: {
        getStatusClass(entry) {
            // TODO: Replace with `entry.status` (will need to pass down)
            if (entry.published && entry.private) {
                return 'bg-transparent border border-gray-600';
            } else if (entry.published) {
                return 'bg-green-600';
            } else {
                return 'bg-gray-400';
            }
        },

        getStatusLabel(entry) {
            if (entry.status === 'published') {
                return __('Published');
            } else if (entry.status === 'scheduled') {
                return __('Scheduled');
            } else if (entry.status === 'expired') {
                return __('Expired');
            } else if (entry.status === 'draft') {
                return __('Draft');
            }
        },

        getStatusTooltip(entry) {
            if (entry.status === 'published') {
                return entry.collection.dated
                    ? __('messages.status_published_with_date', {date: entry.date})
                    : null; // The label is sufficient.
            } else if (entry.status === 'scheduled') {
                return __('messages.status_scheduled_with_date', {date: entry.date})
            } else if (entry.status === 'expired') {
                return __('messages.status_expired_with_date', {date: entry.date})
            } else if (entry.status === 'draft') {
                return null; // The label is sufficient.
            }
        },

        columnShowing(column) {
            return this.cols.find(c => c.field === column);
        },
    }

}
</script>
