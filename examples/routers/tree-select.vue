<template>
    <TreeSelect
        v-model="value"
        :normalizer="normalizer"
        :data="data"
        v-width="200"
        expandNode
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
                    id: '-1',
                    label:'无父级'
                },
                {
                    label: "测试1",
                    id: "parent1",
                    children: [
                        {
                            label: "测试 1-1",
                            expand: true,
                            id: "parent1-1",
                            selected: false,
                            checked: false,
                            children: [
                                {
                                    label: "测试 1-1-1",
                                    id: "leaf1",
                                    selected: false,
                                    checked: false,
                                },
                                {
                                    label: "测试 1-1-2",
                                    id: "leaf2",
                                    selected: false,
                                    checked: false,
                                },
                            ],
                        },
                        {
                            label: "测试 1-2",
                            expand: true,
                            id: "leaf 1-2",
                            selected: false,
                            checked: false,
                            children: [
                                {
                                    label: "测试 1-2-1",
                                    id: "leaf3",
                                    selected: false,
                                    checked: false,
                                },
                                {
                                    label: "测试 1-2-1",
                                    id: "leaf4",
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
        normalizer(node) {
            return {
                title: node.label?node.label:'无父级',
                value: node.id?node.id:'-1',
            };
        },
        handleChange(val) {
            console.log(val);
        },
    },
};
</script>
