<template>
	<div>
		<div class="persian-calendar">
			<div class="calendar-header">
				<button class="arrow-btn" @click="prevMonth">
					<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor"
						class="bi bi-chevron-right" viewBox="0 0 16 16">
						<path fill-rule="evenodd"
							d="M4.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L10.293 8 4.646 2.354a.5.5 0 0 1 0-.708" />
					</svg>
				</button>
				<select class="select-btn" v-model="currentYear">
					<option v-for="year in yearRange" :key="year" :value="year">{{ year }}</option>
				</select>
				<select v-model="currentMonth">
					<option v-for="(month, index) in months" :key="index" :value="index + 1">{{ month }}</option>
				</select>
				<button class="arrow-btn" @click="nextMonth">
					<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor"
						class="bi bi-chevron-left" viewBox="0 0 16 16">
						<path fill-rule="evenodd"
							d="M11.354 1.646a.5.5 0 0 1 0 .708L5.707 8l5.647 5.646a.5.5 0 0 1-.708.708l-6-6a.5.5 0 0 1 0-.708l6-6a.5.5 0 0 1 .708 0" />
					</svg>
				</button>
			</div>
			<div class="calendar-body">
				<div class="day-names">
					<span v-for="day in daysOfWeek" :key="day">{{ day }}</span>
				</div>
				<div class="days">
					<span v-for="(day, index) in paddedDaysInMonth" :key="index"
						:class="{ 'today': isToday(day.shamsiDate), 'disabled': !day.isCurrentMonth, 'selected': isSelected(day.shamsiDate) }"
						@click="selectDate(day)">
						{{ day.date }}
					</span>
				</div>
			</div>
			<div v-if="selectedDate" class="selected-date">
				Selected Date: {{ selectedDateDisplay }}
			</div>
		</div>
	</div>
</template>

<script>
import jalaali from 'jalaali-js';

export default {
	props: {
		value: String,
		options: Object
	},
	data() {
		const today = new Date();
		const shamsiToday = this.toShamsi(today.getFullYear(), today.getMonth() + 1, today.getDate());
		return {
			currentYear: shamsiToday.year,
			currentMonth: shamsiToday.month,
			selectedDate: null,
			selectedDateDisplay: '',
			daysOfWeek: ['شنبه', 'یکشنبه', 'دوشنبه', 'سه‌شنبه', 'چهارشنبه', 'پنج‌شنبه', 'جمعه'],
			months: [
				"فروردین", "اردیبهشت", "خرداد", "تیر", "مرداد", "شهریور",
				"مهر", "آبان", "آذر", "دی", "بهمن", "اسفند"
			],
		};
	},
	computed: {
		daysInMonth() {
			const numberOfDays = this.getDaysInMonth(this.currentYear, this.currentMonth);
			const firstDayOfMonth = this.getFirstDayOfMonth(this.currentYear, this.currentMonth);
			let days = [];

			for (let i = 1; i <= numberOfDays; i++) {
				const shamsiDate = { year: this.currentYear, month: this.currentMonth, day: i };
				days.push({
					date: i,
					shamsiDate: shamsiDate,
					isCurrentMonth: true
				});
			}

			const paddedDays = Array(firstDayOfMonth).fill({ date: '', shamsiDate: null, isCurrentMonth: false }).concat(days);
			const totalCells = 42;
			while (paddedDays.length < totalCells) {
				paddedDays.push({ date: '', shamsiDate: null, isCurrentMonth: false });
			}

			return paddedDays;
		},
		paddedDaysInMonth() {
			return this.daysInMonth;
		},
		yearRange() {
			return this.generateYearRange(1398, 1450);
		}
	},
	methods: {
		toShamsi(gy, gm, gd) {
			let g_d_m = [0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334];
			let jy = (gy <= 1600) ? 0 : 979;
			gy -= (gy <= 1600) ? 621 : 1600;
			let gy2 = (gm > 2) ? (gy + 1) : gy;
			let days = 365 * gy + parseInt((gy2 + 3) / 4) - parseInt((gy2 + 99) / 100) + parseInt((gy2 + 399) / 400) - 80 + gd + g_d_m[gm - 1];
			jy += 33 * parseInt(days / 12053);
			days %= 12053;
			jy += 4 * parseInt(days / 1461);
			days %= 1461;
			if (days > 365) {
				jy += parseInt((days - 1) / 365);
				days = (days - 1) % 365;
			}
			let jm = (days < 186) ? 1 + parseInt(days / 31) : 7 + parseInt((days - 186) / 30);
			let jd = 1 + ((days < 186) ? (days % 31) : ((days - 186) % 30));
			return { year: jy, month: jm, day: jd };
		},
		toGregorian(jy, jm, jd) {
			let gy;
			if (jy > 979) {
				gy = 1600;
				jy -= 979;
			} else {
				gy = 621;
			}

			const j_days_in_month = [31, 31, 31, 31, 31, 31, 30, 30, 30, 30, 30, 29];

			let days = 365 * jy + Math.floor(jy / 33) * 8 + Math.floor((jy % 33 + 3) / 4);
			for (let i = 0; i < jm - 1; ++i) {
				days += j_days_in_month[i];
			}
			days += jd - 1;

			let gy2 = gy + 400 * Math.floor(days / 146097);
			days %= 146097;

			if (days > 36524) {
				gy2 += 100 * Math.floor(--days / 36524);
				days %= 36524;
				if (days >= 365) {
					days++;
				}
			}

			gy2 += 4 * Math.floor(days / 1461);
			days %= 1461;

			if (days > 365) {
				gy2 += Math.floor((days - 1) / 365);
				days = (days - 1) % 365;
			}

			let gd = days + 1;
			const g_d_m = [31, gy2 % 4 == 0 && gy2 % 100 != 0 || gy2 % 400 == 0 ? 29 : 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
			let gm;
			for (gm = 0; gm < 12 && gd > g_d_m[gm]; gm++) {
				gd -= g_d_m[gm];
			}

			return { year: gy2, month: gm + 1, day: gd };
		},
		generateYearRange(start, end) {
			const years = [];
			for (let year = start; year <= end; year++) {
				years.push(year);
			}
			return years; 
		},
		getDaysInMonth(year, month) {
			if (month >= 1 && month <= 6) {
				return 31;
			} else if (month >= 7 && month <= 11) {
				return 30;
			} else if (month === 12) {
				return this.isLeapYear(year) ? 30 : 29;
			}
			return 0;
		},
		isLeapYear(year) {
			// سال شمسی کبیسه است اگر (سال - 474) % 2820 < 682 باشد
			const a = (year + 1 - 474) % 2820;
			return ((((a + 474) * 682) % 2816) < 682);
		},
		getFirstDayOfMonth(year, month) {
			const gregorian = this.toGregorian(year, month, 1);
			const firstDay = new Date(gregorian.year, gregorian.month - 1, gregorian.day).getDay();
			return firstDay;
		},
		isToday(shamsiDate) {
			if (!shamsiDate) return false;
			const today = new Date();
			const shamsiToday = this.toShamsi(today.getFullYear(), today.getMonth() + 1, today.getDate());
			return shamsiDate.year === shamsiToday.year && shamsiDate.month === shamsiToday.month && shamsiDate.day === shamsiToday.day;
		},
		isSelected(shamsiDate) {
			if (!shamsiDate || !this.selectedDate) return false;
			return shamsiDate.year === this.selectedDate.year && shamsiDate.month === this.selectedDate.month && shamsiDate.day === this.selectedDate.day;
		},
		selectDate(day) {
			if (day.isCurrentMonth) {
				this.selectedDate = day.shamsiDate;
				const gregorian = this.toGregorian(day.shamsiDate.year, day.shamsiDate.month, day.shamsiDate.day);
				this.selectedDateDisplay = `${gregorian.year}-${gregorian.month.toString().padStart(2, '0')}-${gregorian.day.toString().padStart(2, '0')}`;
			}
		},
		prevMonth() {
			if (this.currentMonth === 1) {
				this.currentMonth = 12;
				this.currentYear--;
			} else {
				this.currentMonth--;
			}
		},
		nextMonth() {
			if (this.currentMonth === 12) {
				this.currentMonth = 1;
				this.currentYear++;
			} else {
				this.currentMonth++;
			}
		}
	},
	watch: {
		value(newVal) {
			if (newVal) {
				const [year, month, day] = newVal.split('-').map(Number);
				const shamsiDate = this.toShamsi(year, month, day);
				this.selectedDate = shamsiDate;
				this.currentYear = shamsiDate.year;
				this.currentMonth = shamsiDate.month;
				this.selectedDateDisplay = newVal;
			}
		},
		selectedDate(newVal) {
			if (newVal) {
				const gregorian = this.toGregorian(newVal.year, newVal.month, newVal.day);
				this.$emit('input', `${gregorian.year}-${gregorian.month.toString().padStart(2, '0')}-${gregorian.day.toString().padStart(2, '0')}`);
			}
		}
	}
};
</script>

<style scoped>
.persian-calendar {
	width: 100%;
	font-family: 'Vazir', sans-serif;
}

.calendar-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 10px;
}

.select-btn {
	margin: 0 5px;
}

.arrow-btn {
	background: none;
	border: none;
	cursor: pointer;
	font-size: 16px;
}

.calendar-body {
	display: flex;
	flex-direction: column;
}

.day-names {
	display: flex;
	justify-content: space-between;
}

.days {
	display: flex;
	flex-wrap: wrap;
}

.days span {
	width: calc(100% / 7);
	text-align: center;
	padding: 5px;
	cursor: pointer;
	border: 1px solid #ddd;
}

.days span.today {
	background-color: #ffeb3b;
}

.days span.selected {
	background-color: #2196f3;
	color: #fff;
}

.days span.disabled {
	color: #ccc;
	pointer-events: none;
}

.selected-date {
	margin-top: 10px;
}
</style>