<template>
	<v-container>
		<v-app>
			<v-card>
				<v-card-title>
					<v-text-field
						v-model="search"
						append-icon="mdi-magnify"
						label="Поиск"
						single-line
						hide-details
						sort-by="lastOperDt"
						multi-sort
					>
					</v-text-field>
				</v-card-title>
				<v-data-table
					:headers="headers"
					:items="wagons"
					:items-per-page="3"
					item-key="ordNumber"
					:footer-props="{ 'items-per-page-options': opts }"
					class="elevation-1"
					:search="search"
				></v-data-table>
			</v-card>
		</v-app>
	</v-container>
</template>

<script>
import { format } from "date-fns";
import { ru } from "date-fns/locale";

export default {
	data() {
		return {
			opts: [5, 7, 9, 11, 13, -1],
			search: "",
			headers: [
				{
					text: "№ п/п",
					align: "start",
					sortable: true,
					value: "ordNumber",
				},
				{ text: "Номер вагона", value: "carNumber" },
				{ text: "Индекс поезда", value: "trainIndex" },
				{ text: "Номер поезда", value: "trainNumber" },
				{ text: "Статус", value: "carStatus" },
				{ text: "Дата-время операции", value: "lastOperDt" },
				{ text: "№ накладной", value: "invoiceNumber" },
				{ text: "ИД накладной", value: "invoiceId" },
				{ text: "stateId", value: "stateId" },
			],
			wagons: [],
		};
	},
	methods: {
		fetchDate() {
			fetch("http://localhost:5555/table-data.json")
				.then((res) => res.json())
				.then((res) => {
					res.forEach((item) => {
						if (item.lastOperDt) {
							const date = format(
								new Date(item.lastOperDt),
								"dd.MM.yyyy hh:mm",
								{ locale: ru }
							);
							item.lastOperDt = date;
						} else {
							item.lastOperDt = "-";
						}
						this.wagons.push(item);
					});
				});
		},
	},
	mounted() {
		this.fetchDate();
	},
};
</script>
