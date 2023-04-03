<template>
    <div id="burger-table">
        <Message :msg="msg" v-show="msg"/>
        <div>
            <div id="burger-table-heading">
                <div class="order-id">#: </div>
                <div>Cliente: </div>
                <div>Pão: </div>
                <div>Carne: </div>
                <div>Opcionais: </div>
                <div>Ações: </div>
            </div>
            <div id="burger-table-rows">
                <div v-for="pedido in pedidos" :key="pedido.id" class="burger-table-row">
                    <div class="order-number">
                        {{ pedido.id }}
                    </div>
                    <div>{{ pedido.nome }}</div>
                    <div>{{ pedido.pao }}</div>
                    <div>{{ pedido.carne }}</div>
                    <div>
                        <ul>
                            <li v-for="(opcional, index) in pedido.opcionais" :key="index">{{ opcional }}</li>
                        </ul>
                    </div>
                    <div>
                        <select name="status" id="status" @change="updatePedido($event, pedido.id)">
                            <option value="">Selecione</option>
                            <option v-for="status in statusList" :key="status.id" :value="status.tipo" :selected="pedido.status == status.tipo">{{ status.tipo }}</option>
                        </select>
                        <button class="delete-btn" @click="deletePedido(pedido.id)">Cancelar</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import Message from './Message.vue'
    export default{
        name: "Dashboard",
        components: {
            Message
        },
        data(){
            return{
                statusList: [],
                pedidos: null,
                msg: ""
            }
        },
        methods: {
            async getStatus(){
                const status = await fetch("http://localhost:3000/status")
                const statusJSON = await status.json()
                this.statusList = statusJSON
            },
            async getPedidos(){
                const pedidos = await fetch("http://localhost:3000/burgers")
                const pedidosJSON = await pedidos.json()
                this.pedidos = pedidosJSON
            },
            async deletePedido(id){
                const pedido = await fetch(`http://localhost:3000/burgers/${id}`, {
                    method: "DELETE"
                })

                const pedidoJSON = pedido.json()

                this.msg = `Pedido ${id} removido com sucesso!`
                setTimeout(() => {
                    this.msg = ""
                }, 2000);

                this.getPedidos()
            },
            async updatePedido(event, id){
                const option = event.target.value;
                const dataJSON = JSON.stringify({ status: option})
                const pedido = await fetch(`http://localhost:3000/burgers/${id}`, {
                    method: "PATCH",
                    headers: {"Content-Type": "application/json"},
                    body: dataJSON
                })

                const pedidoJSON = await pedido.json()

                console.log(pedidoJSON)

                this.msg = `Pedido ${pedidoJSON.id} atualizado para ${pedidoJSON.status} com sucesso!`
                setTimeout(() => {
                    this.msg = ""
                }, 2000);
            }
        },
        mounted(){
            this.getStatus()
            this.getPedidos()
        }
    }
</script>

<style scoped>
    #burger-table{
        max-width: 1200px;
        margin: 0 auto;
    }

    #burger-table-heading, 
    #burger-table-rows, 
    .burger-table-row{
        display: flex;
        flex-wrap: wrap;
    }

    #burger-table-heading{
        font-weight: bold;
        padding: 12px;
        border-bottom: 3px solid #333;
    }

    #burger-table-heading div,
    .burger-table-row div{
        width: 19%;
    }

    .burger-table-row{
        width: 100%;
        padding: 12px;
        border-bottom: 1px solid #ccc;
    }

    #burger-table-heading .order-id, .burger-table-row .order-number{
        width: 5%;
    }

    select{
        padding: 12px 6px;
        margin-right: 12px;
    }

    .delete-btn{
        background-color: #222;
        color: #fcba03;
        font-weight: bold;
        border: 2px solid #222;
        padding: 10px;
        font-size: 16px;
        margin: 0 auto;
        cursor: pointer;
        transition: .5s;
    }

    .delete-btn:hover{
        background-color: transparent;
        color: #222;
    }
</style>