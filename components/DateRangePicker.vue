<template>
	<v-menu
		:close-on-content-click="false"
		full-width
		lazy
		max-width="850px"
		offset-y
		transition="slide-y-transition"
		v-model="menu"
	>
		<template v-slot:activator="{ on }">
			<v-text-field
				:value="displayText"
				@click:clear="clear"
				clearable
				readonly
				v-bind="$attrs"
				v-on="on"
			></v-text-field>
		</template>

		<v-card>
			<v-layout row wrap>
				<v-flex class="hidden-sm-and-down" xs2>
					<v-card class="pt-5" flat>
						<v-card-text>
							<v-btn
								:key="item.name"
								@click="setDateRange(item)"
								block
								color="primary"
								v-for="item in items"
							>{{item.name}}</v-btn>
						</v-card-text>
					</v-card>
				</v-flex>
				<v-divider flat vertical></v-divider>
				<v-flex>
					<v-card flat>
						<v-card-title class="title secondary white--text justify-center">
							<template v-if="fromTemp && toTemp">
								<b>{{fromTemp}}</b>
								&nbsp;&nbsp;&rarr;&nbsp;&nbsp;
								<b>{{toTemp}}</b>
							</template>
							<template v-else>Select start and end date values</template>
						</v-card-title>
						<v-card-text class="text-xs-center">
							<v-date-picker
								:locale="options.locale"
								:max="toTemp"
								:show-current="false"
								class="elevation-0"
								no-title
								scrollable
								v-model="fromTemp"
							></v-date-picker>
							<v-date-picker
								:locale="options.locale"
								class="elevation-0"
								no-title
								scrollable
								v-model="toTemp"
							></v-date-picker>
						</v-card-text>
						<v-card-actions>
							<v-spacer></v-spacer>
							<v-btn @click="close()" flat>Cancel</v-btn>
							<v-btn :disabled="!(fromTemp && toTemp)" @click="submit()" color="success">OK</v-btn>
						</v-card-actions>
					</v-card>
				</v-flex>
			</v-layout>
		</v-card>
	</v-menu>
</template>

<script>
export default {
  name: 'VDateRangePicker',
  inheritAttrs: false,
  props: {
    value: {
      type: Object,
      default: () => ({})
    },
    from: {
      type: String
    },
    to: {
      type: String
    },
    options: {
      type: Object,
      default: () => ({
        locale: 'en',
        format: 'YYYY-MM-DD'
      })
    }
  },
  data() {
    return {
      menu: false, // Ẩn/hiện menu
      fromTemp: null,
      toTemp: null,
      today: this.$moment(),
      items: [
        {
          name: 'Today',
          number: 0,
          unit: 'days'
        },
        {
          name: 'Yesterday',
          number: -1,
          unit: 'days'
        },
        {
          name: 'This week',
          number: 0,
          unit: 'weeks'
        },
        {
          name: 'Last week',
          number: -1,
          unit: 'weeks'
        },
        {
          name: 'This month',
          number: 0,
          unit: 'months'
        },
        {
          name: 'Last month',
          number: -1,
          unit: 'months'
        },
        {
          name: 'This year',
          number: 0,
          unit: 'years'
        }
      ]
    }
  },
  watch: {
    menu(val) {
      if (val) {
        // Cập nhật biến tạm thời gian
        this.fromTemp = this.from
        this.toTemp = this.to || this.today.format(this.options.format)
      }
    },
    toTemp(val) {
      // Nếu from > to thì xóa from
      if (!this.$moment(this.fromTemp).isSameOrBefore(this.toTemp)) {
        this.fromTemp = null
      }
    }
  },
  computed: {
    // Text hiển thị trong textfield
    displayText() {
      return this.from && this.to ? `${this.from} → ${this.to}` : ''
    }
  },
  methods: {
    // Chọn ngày nhanh
    setDateRange(data) {
      let date = this.today.clone().add(data.number, data.unit)
      let from,
        to = null
      switch (data.unit) {
        case 'days':
          from = to = date
          break
        case 'weeks':
        case 'months':
        case 'years':
          from = date.startOf(data.unit)
          to = date.clone().endOf(data.unit)
          break
      }
      this.fromTemp = from.format(this.options.format)
      this.toTemp = to.format(this.options.format)

      this.submit()
    },

    clear() {
      this.$emit('input', {
        from: null,
        to: null
      })
      this.$emit('update:from', null)
      this.$emit('update:to', null)
    },
    submit() {
      // Cập nhật v-model
      this.$emit('input', {
        from: this.fromTemp,
        to: this.toTemp
      })
      // Sync biến From và To
      this.$emit('update:from', this.fromTemp)
      this.$emit('update:to', this.toTemp)

      this.close()
    },
    close() {
      this.menu = false
    }
  }
}
</script>
<style>
</style>