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
				>
					<template v-slot:top>
						<v-toolbar flat>
							<v-dialog v-model="dialog" max-width="500px">
								<template v-slot:activator="{ on, attrs }">
									<v-btn
										color="primary"
										dark
										class="mb-2"
										v-bind="attrs"
										v-on="on"
									>
										Добавить данные
									</v-btn>
								</template>
								<v-card>
									<v-card-title>
										<span class="text-h5">{{ formTitle }}</span>
									</v-card-title>

									<v-card-text>
										<v-container>
											<v-row>
												<v-col cols="12" sm="6" md="4">
													<v-text-field
														v-model="editedItem.ordNumber"
														label="№ п/п"
													></v-text-field>
												</v-col>
												<v-col cols="12" sm="6" md="4">
													<v-text-field
														v-model="editedItem.carNumber"
														label="Номер вагона"
													></v-text-field>
												</v-col>
												<v-col cols="12" sm="6" md="4">
													<v-text-field
														v-model="editedItem.trainIndex"
														label="Индекс поезда"
													></v-text-field>
												</v-col>
												<v-col cols="12" sm="6" md="4">
													<v-text-field
														v-model="editedItem.trainNumber"
														label="Номер поезда"
													></v-text-field>
												</v-col>
												<v-col cols="12" sm="6" md="4">
													<v-text-field
														v-model="editedItem.carStatus"
														label="Статус"
													></v-text-field>
												</v-col>
												<v-col cols="12" sm="6" md="4">
													<v-text-field
														v-model="editedItem.lastOperDt"
														label="Дата-время операции"
													></v-text-field>
												</v-col>
												<v-col cols="12" sm="6" md="4">
													<v-text-field
														v-model="editedItem.invoiceNumber"
														label="№ накладной"
													></v-text-field>
												</v-col>
												<v-col cols="12" sm="6" md="4">
													<v-text-field
														v-model="editedItem.invoiceId"
														label="ИД накладной"
													></v-text-field>
												</v-col>
												<v-col cols="12" sm="6" md="4">
													<v-text-field
														v-model="editedItem.stateId"
														label="stateId"
													></v-text-field>
												</v-col>
											</v-row>
										</v-container>
									</v-card-text>

									<v-card-actions>
										<v-spacer></v-spacer>
										<v-btn color="blue darken-1" text @click="close">
											Отменить
										</v-btn>
										<v-btn color="blue darken-1" text @click="save">
											Сохранить
										</v-btn>
									</v-card-actions>
								</v-card>
							</v-dialog>
							<v-dialog v-model="dialogDelete" max-width="500px">
								<v-card>
									<v-card-title class="text-h5 text-center"
										>Вы уверены, что хотите удалить этот <br/>элемент?</v-card-title
									>
									<v-card-actions>
										<v-spacer></v-spacer>
										<v-btn color="blue darken-1" text @click="closeDelete"
											>Отменить</v-btn
										>
										<v-btn color="blue darken-1" text @click="deleteItemConfirm"
											>Удалить</v-btn
										>
										<v-spacer></v-spacer>
									</v-card-actions>
								</v-card>
							</v-dialog>
						</v-toolbar>
					</template>

					<template v-slot:item.actions="{ item }">
						<v-icon small class="mr-2" @click="editItem(item)">
							mdi-pencil
						</v-icon>
						<v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
					</template>

					<template v-slot:no-data>
						<v-btn color="primary" @click="fetchDate"> Сбросить </v-btn>
					</template>
				</v-data-table>
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
			lastOrdNumber: 0,
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
				{ text: "Действия", value: "actions", sortable: false },
			],
			wagons: [],
			dialog: false,
			dialogDelete: false,
			editedIndex: -1,
			editedItem: {
				ordNumber: "",
				carNumber: "",
				trainIndex: "",
				trainNumber: "",
				carStatus: "",
				invoiceId: "",
				invoiceNumber: "",
				stateId: "",
				lastOperDt: this.formatDate(new Date()),
			},
			defaultItem: {
				ordNumber: "",
				carNumber: "",
				trainIndex: "",
				trainNumber: "",
				carStatus: "",
				invoiceId: "",
				invoiceNumber: "",
				stateId: "",
				lastOperDt: this.formatDate(new Date()),
			},
		};
	},
	methods: {
		fetchDate() {
			fetch(
				"https://raw.githubusercontent.com/xIgor1982/online-store-api/main/responses/table-data.json"
			)
				.then((res) => res.json())
				.then((res) => {
					res.forEach((item) => {
						item.lastOperDt = item.lastOperDt
							? this.formatDate(item.lastOperDt)
							: "-";
						this.wagons.push(item);
					});
				});
		},

		formatDate(date) {
			const newDate = format(new Date(date), "dd.MM.yyyy hh:mm", {
				locale: ru,
			});
			return newDate;
		},

		editItem(item) {
			this.editedIndex = this.wagons.indexOf(item);
			this.editedItem = Object.assign({}, item);
			this.dialog = true;
		},

		deleteItem(item) {
			this.editedIndex = this.wagons.indexOf(item);
			this.editedItem = Object.assign({}, item);
			this.dialogDelete = true;
		},

		deleteItemConfirm() {
			this.wagons.splice(this.editedIndex, 1);
			this.closeDelete();
		},

		close() {
			this.dialog = false;
			this.$nextTick(() => {
				this.editedItem = Object.assign({}, this.defaultItem);
				this.editedIndex = -1;
			});
		},

		closeDelete() {
			this.dialogDelete = false;
			this.$nextTick(() => {
				this.editedItem = Object.assign({}, this.defaultItem);
				this.editedIndex = -1;
			});
		},

		save() {
			if (this.editedIndex > -1) {
				Object.assign(this.wagons[this.editedIndex], this.editedItem);
			} else {
				this.wagons.push(this.editedItem);
			}
			this.close();
		},
	},
	mounted() {
		this.fetchDate();
	},
	computed: {
		formTitle() {
			return this.editedIndex === -1
				? "Новый элемент"
				: "Редактировать элемент";
		},
	},
	watch: {
		dialog(val) {
			val || this.close();
		},
		dialogDelete(val) {
			val || this.closeDelete();
		},
	},
};
</script>
