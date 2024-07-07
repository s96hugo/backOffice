<template>
  <div>
    <div class="filter-container">
      <label for="screenTypeFilter">Filtrar por: </label>
      <select id="screenTypeFilter" v-model="screenType">
        <option value="1">Barra</option>
        <option value="2">Cocina</option>
      </select>
    </div>
    <div class="tables-container">
      <div class="table-section">
        <h3>Pendiente</h3>
        <table>
          <thead>
            <tr>
              <th>Nombre</th>
              <th>Cantidad</th>
              <th>Mesa</th>
              <th>Pedido</th>
              <th>En preparación</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="ticket in pendingTickets" :key="ticket.numeroPedido">
              <td>
                {{ ticket.nombre }}
                <div v-if="ticket.comment">
                  <br />
                  <strong>Comentario: {{ ticket.comment }}</strong>
                </div>
              </td>
              <td>{{ ticket.cantidad }}</td>
              <td>{{ ticket.mesa }}</td>
              <td>{{ ticket.numeroPedido }}</td>
              <td>
                <button @click="changeTicketStatus(ticket, 2)">➡️</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="table-section">
        <h3>En preparación</h3>
        <table>
          <thead>
            <tr>
              <th>Nombre</th>
              <th>Cantidad</th>
              <th>Mesa</th>
              <th>Pedido</th>
              <th>Pentiente</th>
              <th>Finalizado</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="ticket in inProgressTickets" :key="ticket.numeroPedido">
              <td>
                {{ ticket.nombre }}
                <div v-if="ticket.comment">
                  <br />
                  <strong>Comentario: {{ ticket.comment }}</strong>
                </div>
              </td>
              <td>{{ ticket.cantidad }}</td>
              <td>{{ ticket.mesa }}</td>
              <td>{{ ticket.numeroPedido }}</td>
              <td>
                <button @click="changeTicketStatus(ticket, 1)">⬅️</button>
              </td>
              <td>
                <button @click="changeTicketStatus(ticket, 3)">➡️</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="table-section">
        <h3>Finalizado</h3>
        <table>
          <thead>
            <tr>
              <th>Nombre</th>
              <th>Cantidad</th>
              <th>Mesa</th>
              <th>Pedido</th>
              <th>En preparación</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="ticket in completedTickets" :key="ticket.numeroPedido">
              <td>
                {{ ticket.nombre }}
                <div v-if="ticket.comment">
                  <br />
                  <strong>Comentario: {{ ticket.comment }}</strong>
                </div>
              </td>
              <td>{{ ticket.cantidad }}</td>
              <td>{{ ticket.mesa }}</td>
              <td>{{ ticket.numeroPedido }}</td>
              <td>
                <button @click="changeTicketStatus(ticket, 2)">⬅️</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      tickets: [], // Almacena los datos de los tickets para la tabla
      screenType: '1' // Valor por defecto del combobox (Cocina)
    };
  },
  created() {
    this.fetchTickets(); // Llama a fetchTickets cuando se crea el componente
    this.setupAutoRefresh(); // Configura la actualización automática
  },
  computed: {
    filteredTickets() {
      return this.tickets.filter(ticket => ticket.screenType == this.screenType); // Filtra tickets según el valor del screenType seleccionado
    },
    pendingTickets() {
      return this.filteredTickets.filter(ticket => ticket.status == 1);
    },
    inProgressTickets() {
      return this.filteredTickets.filter(ticket => ticket.status == 2);
    },
    completedTickets() {
      return this.filteredTickets.filter(ticket => ticket.status == 3);
    }
  },
  methods: {
    async fetchTickets() {
      const authToken = localStorage.getItem('authToken'); // Obtener el token de autenticación del localStorage
      try {
        const response = await axios.get('http://192.168.1.133:8081/api/ticketsOpen', {
          headers: {
            'Authorization': `Bearer ${authToken}` // Pasar el token en el encabezado de la solicitud
          }
        });

        if (response.data.success && response.data.tickets) {
          const tickets = response.data.tickets;

          // Crear un array de promesas para realizar solicitudes adicionales por cada ticket
          const ticketPromises = tickets.map(async (ticket) => {
            const productResponse = await axios.get(`http://192.168.1.133:8081/api/products/${ticket.id}/dataSet`, {
              headers: {
                'Authorization': `Bearer ${authToken}` // Pasar el token en el encabezado de la solicitud
              }
            });

            // Suponiendo que productResponse.data.ticketOrderInfo es el array de información de pedidos
            const ticketOrderInfo = productResponse.data.ticketOrderInfo;
            // Mapear la información del pedido al formato requerido
            return ticketOrderInfo.map(order => ({
              id: order.id,
              nombre: order.name,
              cantidad: order.units,
              mesa: ticket.table_id,
              numeroPedido: order.order_id,
              screenType: order.screenType, // Asumimos que screenType está en la respuesta de order
              status: order.status, // Asumimos que status está en la respuesta de order
              comment: order.comment // Asumimos que comment está en la respuesta de order
            }));
          });

          // Esperar a que todas las promesas se resuelvan y aplanar el resultado
          const resolvedTickets = await Promise.all(ticketPromises);
          this.tickets = resolvedTickets.flat(); // Asignar los datos a la propiedad tickets
        }
      } catch (error) {
        console.error('Error fetching tickets:', error); // Manejar errores
      }
    },
    async changeTicketStatus(ticket, status) {
      const authToken = localStorage.getItem('authToken'); // Obtener el token de autenticación del localStorage
      const url = `http://192.168.1.133:8081/api/productOrders/${status}/status`;
      const data = { order_id: ticket.id };

      try {
        const response = await axios.post(url, data, {
          headers: {
            'Authorization': `Bearer ${authToken}` // Pasar el token en el encabezado de la solicitud
          }
        });

        if (response.data.success) {
          ticket.status = status; // Actualizar el estado del ticket localmente si la solicitud fue exitosa
        }
      } catch (error) {
        console.error(`Error changing ticket status to ${status}:`, error); // Manejar errores
      }
    },
    setupAutoRefresh() {
      setInterval(() => {
        this.fetchTickets();
      }, 8000); // Actualizar cada 8 segundos
    }
  }
};
</script>

<style>
.filter-container {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 10px;
}

.tables-container {
  display: flex;
  justify-content: space-between;
}

.table-section {
  flex: 1;
  margin: 0 10px;
}

table {
  width: 100%;
  border-collapse: collapse; /* Colapsar bordes de la tabla */
}

th, td {
  border: 1px solid #ccc; /* Bordes de celdas */
  padding: 8px; /* Espaciado de celdas */
  text-align: left; /* Alineación de texto a la izquierda */
}

th {
  background-color: #f4f4f4; /* Color de fondo de encabezados */
}

h3 {
  text-align: center;
  margin-bottom: 10px;
}

button {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1.2em; /* Aumentar el tamaño de la fuente */
}
</style>
