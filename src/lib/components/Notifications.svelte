<!-- Logic -->
 
<script module>

	/**
	 * @typedef {Object} Notification
	 * @property {number} id
	 * @property {string} message
	*/
	
	const notificationList = $state([]);

	/**
	 * Creates a new notification and adds it to the notification list.
	 * @param {string} message The message to create the notification with.
	 * @returns {Notification} The notification that was created.
	 */
	export function pushNotif(message) {
		const id = Date.now();

		/** @type { Notification } */
		const newNotif = {id, message};

		notificationList.push(newNotif);
		setTimeout(() => pullNotif(newNotif), 5000);

		return newNotif;
	}

	/**
	 * Removes a given notification from the notification list if it's still there.
	 * @param { Notification } notifObject The notification to remove.
	 */
	export function pullNotif(notifObject) {
		const index = notificationList.findIndex(
			(iteratedNotif) => {
				return iteratedNotif.id === notifObject.id;
			}
		);
		
		if (index != -1) {
			notificationList.splice(index, 1);
		}
	}
	
	/**
	 * Removes all notifications from the notification list.
	 */
	export function clearNotifs() {
		notificationList.length = 0;
	}

</script>


<!-- Markup -->

<div class="container">
	{#each notificationList as notif}
		<div class="popup">
			{notif.message}
		</div>
	{/each}
</div>


<!-- Styles -->

<style>
	.container {
		position: fixed;
		top: 1rem;
		right: 1rem;
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.popup {
		background: #ff4d4f;
		color: white;
		padding: 0.75rem 1rem;
		border-radius: 6px;
		box-shadow: 0 2px 8px rgba(0,0,0,0.2);
		font-size: 0.9rem;
	}
</style>
