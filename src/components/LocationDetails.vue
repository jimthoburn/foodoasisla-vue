<template>
	<div class="location-summary-container">
		<div v-bind:class="[location.categoryCode, location.parentCategoryCode]">
			<!--
			<p class="action" style="text-align: center; font-size: 0.75em; margin-top: -0.375rem; margin-bottom: 0; display: none;">
				<button type="button">
					Close
				</button>
			</p>
			-->

	        <div class="location-summary">
				<img v-bind:src="'/assets/images/home/' + location.categoryCode + '.svg'" width="100" alt="" />
				<h2>{{ location.name }}</h2>
	            <p class="address">{{ location.address_1 }}<br />{{ location.address_2 }}</p>
	            <p class="type">{{ location.category }}</p>
	            <p v-if="isOpenNow" class="open">Open Now</p>
	            <p class="distance"><span>{{ distance }}</span> <abbr title="miles">mi</abbr></p>
	        </div>

			<ul class="options action">
				<li><a href="#shareable-link" v-on:click.prevent="shareableLinkActive = true"><span><img src="/assets/images/icons/share.svg" height="24" class="icon" alt="" /></span> <span>Share</span></a></li>
				<li><a href="#directions" v-on:click.prevent="directionsActive = true"><span><img src="/assets/images/icons/directions.svg" height="24" class="icon" alt="" /></span> <span>Directions</span></a></li>
			</ul><!-- /.options -->

			<div class="shareable-link" id="shareable-link" v-show="shareableLinkActive">
				<h2>Share</h2>
				<p class="copy-paste">
					<label>
						<span>Here’s a link you can copy and paste:</span>
						<input type="text" v-bind:value="shareableLink" onclick="this.setSelectionRange(0, this.value.length)" readonly="readonly" />
					</label>
				</p>
			</div><!-- /.shareable-link -->

			<div class="directions" id="directions" v-show="directionsActive">
				<h2>Directions</h2>
				<p class="copy-paste">
					<label>
						<span>Here’s an address you can copy and paste:</span>
						<textarea onclick="this.setSelectionRange(0, this.value.length)" readonly="readonly">{{ directionsPasteable }}</textarea>
					</label>
				</p>
				
				<p><a v-bind:href="directionsURL">Get Directions on Google Maps</a></p>
			</div>

			<section class="dates" v-if="location.season_open && location.season_close">
				<h2>Dates</h2>
				<p>{{ location.season_open }} - {{ location.season_close }}</p>
			</section>

			<section class="hours">
				<h2>Hours</h2>
				<dl>
					<template v-for="item in location.hours">
						<dt>{{ getFormattedWeekday(item) }}</dt>
						<dd v-if="isOpenNowOnWeekday(item)" class="open"><span>{{ getFormattedHours(item) }}</span> <i>Open Now</i></dd>
						<dd v-else-if="item.open"><span>{{ getFormattedHours(item) }}</span></dd>
						<dd v-else><i>Closed</i></dd>
					</template>
				</dl>
			</section>

			<section class="information">
				<h2>Information</h2>
				<dl>
					<dt class="website" v-if="location.website">Website</dt>
					<dd class="website" v-if="location.website"><a v-bind:href="location.website">{{ location.website }}</a></dd>

					<dt class="phone"   v-if="location.phone">Phone</dt>
					<dd class="phone"   v-if="location.phone">{{ location.phone }}</dd>
				</dl>
			</section>

			<div class="content" v-html="location.content"></div>

			<section class="note">
				<h2>This location’s address and hours may have changed.</h2> 
				<p>{{ confirmationNote }}</p>
			</section>

			<section class="location-details-options">
				<h2>Options</h2>

				<ul class="options action secondary">
					<li><a v-bind:href="claimBusinessURL">Claim Business</a></li>
					<li><a v-bind:href="reportIssueURL">Report Issue</a></li>
				</ul>
			</section>
		</div>
	</div>
</template>

<script>
export default {
	props: {
		location: {
			type: Object,
			required: true
		}
	},
	data: function() {
		return {
			shareableLinkActive: false,
			directionsActive: false
		}
	},
	methods: {
		getFormattedWeekday: function(hours) {
			switch (hours.day.trim().toLowerCase()) {
				case 'mon':
					return 'Monday';
				case 'tue':
					return 'Tuesday';
				case 'wed':
					return 'Wednesday';
				case 'thu':
					return 'Thursday';
				case 'fri':
					return 'Friday';
				case 'sat':
					return 'Saturday';
				case 'sun':
					return 'Sunday';
			}

			return hours.day
		},

		isOpenNowOnWeekday: function (weekdayHours) {
			return window.oasis.isOpenNow(weekdayHours)
		},

		getFormattedHours: function(hours) {
			return this.formatTime(hours.open) + ' – ' + this.formatTime(hours.close)
		},

		formatTime: function(timeString) { // Example: 1430 ==> 2:30pm; 0900 ==> 9:00am
			let hours   = Number(timeString.substring(0, timeString.length - 2));
			let minutes = timeString.substring(timeString.length - 2);
			let ampm = 'am';
			if (hours >= 12 && hours < 24) {
				ampm = 'pm';
			}
			if (hours >= 13) {
				hours = hours - 12;
			}
			return hours + ((minutes != '00') ? ":" + minutes : '') + ampm;
		}
	},
	computed: {
		distance: function () {
			return window.oasis.getDistanceForPresentation(this.location.distanceFromYou)
		},
		confirmationNote: function () {
			// Add a message encouraging the visitor to call ahead before visiting a location.
			if (this.location.phone != '') {
				return `
					Before you visit, please call this location’s phone number: ${this.location.phone} and ask them for their address and hours.
				`;
			} else {
				return `
					Before you visit, please contact this location and ask them for their address and hours.
				`;
			}
		},
		shareableLink: function () {
			return 'https://foodoasis.la' + this.location.uri
		},
		directionsPasteable: function () {
			let directionsPasteable = 
				this.location.name      + '\r\n' +
				this.location.address_1 + '\r\n';

			if (this.location.address_2 && this.location.address_2 !== '') {
				directionsPasteable += this.location.address_2 + '\r\n';
			}

			directionsPasteable += this.location.city + ', California ' + this.location.zip;

			return directionsPasteable;
		},
		directionsURL: function () {
			let query = 
				this.location.latitude + ',' + 
				this.location.longitude + ',' + 
				this.location.name + ' ' + 
				this.location.address_1;

			if (this.location.address_2 && this.location.address_2 !== '') {
				query += ' ' + this.location.address_2
			}

			query += ' ' + this.location.city + ', California ' + this.location.zip

			return 'https://maps.google.com/maps?q=' + encodeURIComponent(query)
		},
		claimBusinessURL: function () {
			let subject = `Claim Business, ${ location.name }, Food Oasis LA&body=Hello team at Food Oasis LA. I’d like to claim this business… ${ location.name }: https://foodoasis.la${ location.uri }`

			return 'mailto:contact@foodoasis.la?subject=' + encodeURIComponent(subject)
		},
		reportIssueURL: function () {
			let params = {
				businessName: this.location.name,
				address:      this.location.address,
				category:     this.location.category,
				longitude:    this.location.longitude,
				latitude:     this.location.latitude
			}

			let queryString = []
			for (var key in params) {
				queryString.push(key + '=' + encodeURIComponent(params[key]))
			}

			return 'https://form.jotform.com/62638504761156?' + queryString.join('&')
		},
		isOpenNow: function () {
			for (let index = 0; index < this.location.hours.length; index++) {
				if (window.oasis.isOpenNow(this.location.hours[index])) {
					return true
				}
			}
			return false
		}
	}
}
</script>
