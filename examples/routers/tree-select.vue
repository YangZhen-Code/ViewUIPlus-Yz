<template>
    <TreeSelect
        v-model="value"
        multiple
        showCheckbox
        :data="data"
        v-width="200"
        @on-change="handleChange"
    />
</template>
<script>
export default {
    data() {
        return {
            value: "",
            data: [
                {
                    title: "测试1",
                    expand: true,
                    value: "parent1",
                    selected: false,
                    checked: false,
                    children: [
                        {
                            title: "测试 1-1",
                            expand: true,
                            value: "parent1-1",
                            selected: false,
                            checked: false,
                            children: [
                                {
                                    title: "测试 1-1-1",
                                    value: "leaf1",
                                    selected: false,
                                    checked: false,
                                },
                                {
                                    title: "测试 1-1-2",
                                    value: "leaf2",
                                    selected: false,
                                    checked: false,
                                },
                            ],
                        },
                        {
                            title: "测试 1-2",
                            expand: true,
                            value: "leaf 1-2",
                            selected: false,
                            checked: false,
                            children: [
                                {
                                    title: "测试 1-2-1",
                                    value: "leaf3",
                                    selected: false,
                                    checked: false,
                                },
                                {
                                    title: "测试 1-2-1",
                                    value: "leaf4",
                                    selected: false,
                                    checked: false,
                                },
                            ],
                        },
                    ],
                },
            ],
        };
    },
    methods: {
        handleChange(val) {
            console.log(val);
        },
    },
};
</script>
