<template>
    <!-- Main Content -->
    <div class="content">
        <div class="left-panel">
            <!-- Component 1: Folder Navigation -->
            <div class="folder-navigation">
                <DatabaseDropdown />
            </div>
            <!-- Component 2: Column Navigation -->
            <div class="column-navigation">
                <ColumnDropdown :selectedTable="selectedTable" />
            </div>
        </div>

        <!-- Component 3: Selection, Interval, Aggregation, and Export Config -->
        <div class="settings-panel">
            <Selection />
            <IntervalDropdown />
            <StatisticsDropdown />
            <AggregationMethod />
            <ExportConfig />
            <button class="fetch-button" @click="fetchData">Fetch Data</button>
            <button class="export-button" @click="exportData">Export Data</button>
            <h4 v-if="exportFilename">Saved {{ exportFilename }} to {{ exportPath }} as a {{ exportFormat }}</h4>
        </div>

        <!-- Component 4: Main View (Table and Stats Display) -->
        <div class="main-view">
            <!-- Table Container with Scrollable Body -->
            <div class="table-container">
                <table class="styled-table">
                    <thead>
                        <tr>
                            <th v-for="column in selectedColumns" :key="column">{{ column }}</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(row, index) in visibleData" :key="index">
                            <td v-for="column in selectedColumns" :key="column">{{ row[column] }}</td>
                        </tr>
                    </tbody>
                </table>
                <!-- Load More Button -->
                <div v-if="canLoadMore" class="load-more-container">
                    <button class="load-more-button" @click="loadMoreRows">Load More</button>
                </div>
            </div>
            <!-- Stats Container with Scrollable Body -->
            <div class="stats-container">
                <table class="styled-table">
                    <thead>
                        <tr>
                            <th v-for="column in statsColumns" :key="column">{{ column }}</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(row, index) in stats" :key="index">
                            <td v-for="column in statsColumns" :key="column">{{ row[column] }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script>
import { mapState, mapActions } from "vuex";
import DatabaseDropdown from "../components/DatabaseDropdown.vue";
import ColumnDropdown from "../components/ColumnDropdown.vue";
import Selection from "../components/Selection.vue";
import IntervalDropdown from "../components/IntervalDropdown.vue";
import AggregationMethod from "../components/AggregationMethod.vue";
import StatisticsDropdown from "../components/StatisticsDropdown.vue";
import ExportConfig from "../components/ExportConfig.vue";
import axios from 'axios';

export default {
    name: "Project",
    components: {
        DatabaseDropdown,
        ColumnDropdown,
        Selection,
        IntervalDropdown,
        AggregationMethod,
        StatisticsDropdown,
        ExportConfig,
    },
    data() {
        return {
            stats: [],
            statsColumns: [],
            visibleData: [],
            rowLimit: 100,
            canLoadMore: true,
            data: [],
        };
    },
    computed: {
        ...mapState(["selectedDb", "selectedTable", "selectedColumns", "selectedIds", "dateRange", "selectedInterval", "selectedStatistics", "selectedMethod", "exportColumns", "exportIds", "exportDate", "exportInterval", "dateType", "exportDateType", "exportPath", "exportFilename", "exportFormat", "exportOptions"]),
    },
    methods: {
        ...mapActions(["updateSelectedColumns"]),
        selectFolder() {
            // Placeholder for selecting a folder, could be integrated with backend logic
            this.folderPath = "Jenette_Creek_Watershed";
        },
        loadInitialRows() {
            this.visibleData = this.data.slice(0, this.rowLimit);
            this.canLoadMore = this.data.length > this.rowLimit;
        },
        loadMoreRows() {
            const nextRowLimit = this.visibleData.length + this.rowLimit;
            const nextRows = this.data.slice(this.visibleData.length, nextRowLimit);
            // Append the next rows to the visible data
            this.visibleData = [...this.visibleData, ...nextRows];
            // Check if we can load more rows
            if (this.visibleData.length >= this.data.length) {
                this.canLoadMore = false; // Hide the load more button
            }
        },
        async fetchData() {
            try {
                const response = await axios.get(`${import.meta.env.VITE_APP_API_BASE_URL}/api/get_data`, {
                    params: {
                        db_path: this.selectedDb,
                        table_name: this.selectedTable,
                        columns: this.selectedColumns.filter((column) => column !== 'Season').join(","),
                        id: this.selectedIds.join(","),
                        start_date: this.dateRange.start,
                        end_date: this.dateRange.end,
                        date_type: this.dateType,
                        interval: this.selectedInterval,
                        statistics: this.selectedStatistics.join(","),
                        method: this.selectedMethod.join(","),
                    }
                });
                if (this.selectedInterval === 'seasonally' && !this.selectedMethod.includes('Equal') && !this.selectedColumns.includes('Season')) {
                    this.updateSelectedColumns(this.selectedColumns.concat(['Season']));
                } else{
                    this.updateSelectedColumns(this.selectedColumns.filter((column) => column !== 'Season'));
                }
                this.data = response.data.data;
                this.stats = response.data.stats;
                this.statsColumns = response.data.statsColumns;
                this.loadInitialRows();
            } catch (error) {
                console.error('Error fetching data:', error);
            }
        },
        async exportData() {
            try {
                await axios.get(`${import.meta.env.VITE_APP_API_BASE_URL}/api/export_data`, {
                    params: {
                        db_path: this.selectedDb,
                        table_name: this.selectedTable,
                        columns: this.exportColumns.filter((column) => column !== 'Season').join(","),
                        id: this.exportIds.join(","),
                        start_date: this.exportDate.start,
                        end_date: this.exportDate.end,
                        date_type: this.exportDateType,
                        interval: this.exportInterval,
                        statistics: this.selectedStatistics.join(","),
                        method: this.exportMethod,
                        export_path: this.exportPath,
                        export_filename: this.exportFilename,
                        export_format: this.exportFormat,
                        options: this.exportOptions,
                    }
                });
                if (this.selectedInterval === 'seasonally' && !this.selectedMethod.includes('Equal') && !this.selectedColumns.includes('Season')) {
                    this.updateSelectedColumns(this.selectedColumns.concat(['Season']));
                } else{
                    this.updateSelectedColumns(this.selectedColumns.filter((column) => column !== 'Season'));
                }
            } catch (error) {
                console.error('Error exporting data:', error);
            }
        },
    },
};
</script>

<style scoped>
body,
html {
    margin: 0;
    padding: 0;
    height: 100%;
    overflow: hidden;
    /* Ensure no scroll at the global level */
}

.content {
    display: flex;
    flex-direction: row;
    height: calc(100vh - 130px);
    /* Adjust to the viewport, minus top bar and taskbar */
}

.left-panel {
    display: flex;
    flex-direction: column;
    width: 25%;
    /* Takes 1/4 of the horizontal space */
    height: 100%;
    /* Ensure full height */
    overflow: hidden;
}

.folder-navigation {
    margin: 0;
    border: 1px solid #ddd;
    padding: 10px;
    height: 35%;
    /* Takes 1/4 of the vertical space */
    overflow-y: auto;
}

.column-navigation {
    margin: 0;
    border: 1px solid #ddd;
    padding: 0px;
    height: 85%;
    /* Takes the remaining vertical space */
    overflow-y: auto;
}

.settings-panel {
    width: 50%;
    /* Takes 2/4 of the horizontal space */
    height: 100%;
    /* Ensure full height */
    margin: 10px;
    border: 1px solid #ddd;
    padding: 10px;
    overflow-y: auto;
}

.main-view {
    width: 50%;
    /* Takes 2/4 of the horizontal space */
    height: 100%;
    /* Ensure full height */
    margin: 0px;
    border: 1px solid #ddd;
    padding: 10px;
    overflow-y: auto;
}

.table-container,
.stats-container {
    max-width: 100%;
    max-height: 300px;
    overflow-y: auto;
    border: 1px solid #ddd;
}

.styled-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 14px;
    text-align: left;
}

.styled-table thead {
    position: sticky;
    top: 0;
    z-index: 2;
    background-color: #9e3203;
    color: #ffffff;
}

.styled-table th,
.styled-table td {
    padding: 12px 15px;
    border: 1px solid #dddddd;
}

.styled-table tbody tr:nth-of-type(even) {
    background-color: #f3f3f3;
}

.styled-table tbody tr:hover {
    background-color: #f1f1f1;
}

.fetch-button,
.export-button {
    background-color: #009879;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin-top: 20px;
}

.load-more-button {
    background-color: #009879;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}
</style>