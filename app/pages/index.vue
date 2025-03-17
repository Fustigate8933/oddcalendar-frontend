<template>
	<div class="flex flex-col items-center justify-center gap-4 h-screen">
		<h1 class="font-bold text-2xl text-(--ui-secondary)">
			Odd Calendar
		</h1>

		<div class="flex items-center gap-2">
			<input
				ref="fileInput"
				type="file"
				accept=".html"
				class="hidden"
				@change="handleFileUpload"
			>

			<UButton
				label="Upload HTML"
				color="neutral"
				variant="outline"
				icon="material-symbols:upload-file-outline"
				target="_blank"
				class="cursor-pointer"
				@click="triggerFileInput"
			/>

			<UButton
				label="Download ics"
				icon="i-lucide-square-play"
				target="_blank"
				class="cursor-pointer"
				:disabled="!htmlContent"
				@click="downloadICS"
			/>
		</div>
		<ol class="list-decimal">
			<li>Nagivate to <NuxtLink class="underline text-(--ui-warning)" to="https://odyssey.uwaterloo.ca/teaching/schedule">Odyssey</NuxtLink></li>
			<li>Right click on the page</li>
			<li>Select "<span class="text-rose-200">Save Page As...</span>"</li>
			<li>Upload the download file</li>
			<li>Download your calendar file!</li>
		</ol>  
	</div>
</template>

<script setup>
const fileInput = ref(null);
const htmlContent = ref(null);

const triggerFileInput = () => {
	fileInput.value.click();
}

const handleFileUpload = (event) => {
	const file = event.target.files[0];
	if (!file) return;

	const reader = new FileReader();
	reader.onload = (e) => {
		htmlContent.value = e.target.result;
	};
	reader.readAsText(file);
};

const downloadICS = async () => {
	if (!htmlContent.value) return;

	console.log(htmlContent.value);

	try {
		// const uri = "https://oddcalendar-backend.onrender.com/download-calendar";
		const uri = "http://127.0.0.1:8000/download-calendar";
		const response = await fetch(uri, {
			method: "POST",
			headers: {
				"Content-Type": "application/json",
			},
			body: JSON.stringify({ html: htmlContent.value }),
		});

		if (!response.ok) {
			throw new Error("Failed to fetch ICS file");
		}

		const blob = await response.blob();
		const url = window.URL.createObjectURL(blob);

		const a = document.createElement("a");
		a.href = url;
		a.download = "odyssey_schedule.ics";
		document.body.appendChild(a);
		a.click();
		document.body.removeChild(a);

		window.URL.revokeObjectURL(url);
	} catch (error) {
		console.error("Error:", error);
	}
};
</script>
