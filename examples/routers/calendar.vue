<template>
    <div>
        <Calendar
            v-model="value"
            type="year"
            multiple
            :defaultList="defaultList"
            hide-type
            @on-prev="handlePrev"
            @on-select="handleSelect"
        >
            <template #year="{ data }">
                <Badge
                    status="warning"
                    text="看苹果发布会"
                    v-if="data.month === '2024-09'"
                />
                <Badge
                    status="success"
                    text="中秋节"
                    v-if="data.day === '2019-09-13'"
                />
                <Badge
                    status="error"
                    text="国庆节"
                    v-if="data.day === '2019-10-01'"
                />
            </template>
        </Calendar>
    </div>
</template>
<script>
export default {
    data() {
        return {
            value: "2024-09",
            defaultList: ['2024-01'],
            activeList: ["2024-10"],
        };
    },
    methods: {
        handlePrev(val) {
            console.log(val);
        },
        handleSelect(val) {
            console.log(val);
        }
    },
};
</script>
<style>
.ivu-calendar-table-day-disabled {
    pointer-events: none;
}
</style>
