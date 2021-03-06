<template>
    <layout>
      <template v-slot:header>
        <simple-header title="Gastos" button-text="Crear gasto" button-link='/expenses/create' />
      </template>
      <template v-slot:content>
        <simple-table :headings="headings" :data="expenses" :loading="loading" @editRecord="editRecord" @deleteRecord="deleteRecord"></simple-table>
      </template>
    </layout>
</template>

<script>
    import moment from 'moment';
    import EXPENSES from '../../graphql/expenses/expenses.graphql';
    import DELETE_EXPENSE from '../../graphql/expenses/delete-expense.graphql';
    import Swal from 'sweetalert2/dist/sweetalert2.js'
    import 'sweetalert2/src/sweetalert2.scss'

    export default {
        data() {
            return {
                headings: [
                    'Fecha',
                    'Categoría',
                    'Medio de pago',
                    'Descripción',
                    'Valor'
                ],
                expenses: [],
                loading: true
            }
        },
        mounted() {
            this.getExpenses();
        },
        methods: {
            async getExpenses() {
                const response = await this.$apollo.query({
                    query: EXPENSES,
                    variables: {
                      first: 20
                    }
                });
                this.expenses = response.data.expenses.edges.map(item => {
                    return {
                        id: item.node.id,
                        date: moment(item.node.date).format('MM/DD/YYYY'),
                        category: item.node.category.name,
                        paymentMethod: item.node.paymentMethod.name,
                        description: item.node.description,
                        value: new Intl.NumberFormat('es-CO', { style: 'currency', currency: 'COP' }).format(item.node.amount),
                    };
                });
                this.loading = this.$apollo.loading;
            },
            editRecord(record) {
                this.$router.push(`/expenses/${record.id}/edit`);
            },
            deleteRecord(record) {
                Swal.fire({
                    title: 'Estas seguro?',
                    text: `Quieres eliminar el gasto?, esto no es reversible.`,
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Si, eliminar!'
                }).then((result) => {
                    if (result.value) {
                        this.loading = true;
                        return this.$apollo.mutate({
                            mutation: DELETE_EXPENSE,
                            variables: {
                                id: record.id
                            }
                        });
                    }
                }).then(response => {
                    this.$toasted.success('Gasto eliminado satisfactoriamente!', {
                        position: "top-center",
                        duration : 5000
                    });
                    this.getExpenses();
                })
            }
        }
    }
</script>

<style scoped>

</style>
