<template>
    <div>
        <h1>Mortgage table</h1>
        <h2>{{ userName }}</h2>
        <Table class="display" width="100%">
            <thead>
                <tr>
                    <th>Fixation length</th>
                    <th>Monthly rate</th>
                    <th>Interest rate</th>
                </tr>
            </thead>
            <!-- <tbody>
                <tr v-for="data in mortgageList" :key="data.borrowingRate">
                    <td>{{ data.borrowingRate }}</td>
                    <td>{{ data.monthlyRate }}</td>
                </tr>
            </tbody> -->
        </Table>
    </div>
</template>

<script>
const props = defineProps(['userName'])
export default {
    name: 'MortgageTable',
    props: {
        userName
    }
}

</script>