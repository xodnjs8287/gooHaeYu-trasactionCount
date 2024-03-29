<script lang="ts">
	import { page } from '$app/stores';
	import rq from '$lib/rq/rq.svelte';

	async function loadApplication() {
		const { data } = await rq.apiEndPoints().GET('/api/applications/{id}', {
			params: {
				path: {
					id: parseInt($page.params.id)
				}
			}
		});

		return data!.data!;
	}

	function formatDate(dateString) {
		const options = {
			year: '2-digit',
			month: '2-digit',
			day: '2-digit',
			hour: '2-digit',
			minute: '2-digit'
		};
		return new Date(dateString).toLocaleString('ko-KR', options);
	}

	function calculateAge(birthDate) {
		const birth = new Date(birthDate);
		const today = new Date();
		let age = today.getFullYear() - birth.getFullYear();
		const m = today.getMonth() - birth.getMonth();
		if (m < 0 || (m === 0 && today.getDate() < birth.getDate())) {
			age--;
		}
		return age;
	}

	async function deleteApplication(applicationId: number) {
		try {
			const response = await rq.apiEndPoints().DELETE(`/api/applications/${applicationId}`, {
				headers: { 'Content-Type': 'application/json' }
			});

			if (response.data?.msg == 'CUSTOM_EXCEPTION') {
				alert(response.data?.data?.message);
			}

			if (response.data?.statusCode === 204) {
				alert('지원서가 삭제되었습니다.');
				rq.goTo('/member/me');
			}
		} catch (error) {
			alert('지원서 삭제에 실패했습니다. 다시 시도해주세요.');
		}
	}

	function goToEditPage(applicationId: number) {
		rq.goTo(`/applications/modify/${applicationId}`);
	}
</script>

{#await loadApplication() then application}
	<div class="flex justify-center items-center min-h-screen bg-base-100">
		<div class="container mx-auto px-4 mt-5">
			<div class="w-full max-w-4xl mx-auto">
				<div class="card bg-base-100 shadow-xl rounded-lg p-5">
					<div class="flex items-center mb-4">
						<svg
							xmlns="http://www.w3.org/2000/svg"
							fill="none"
							viewBox="0 0 24 24"
							stroke-width="1.5"
							stroke="currentColor"
							class="w-6 h-6 mr-2"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								d="M19.5 14.25v-2.625a3.375 3.375 0 0 0-3.375-3.375h-1.5A1.125 1.125 0 0 1 13.5 7.125v-1.5a3.375 3.375 0 0 0-3.375-3.375H8.25m0 12.75h7.5m-7.5 3H12M10.5 2.25H5.625c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125V11.25a9 9 0 0 0-9-9Z"
							/>
						</svg>
						<h2 class="card-title text-xl font-bold">지원서 상세 정보</h2>
					</div>

					<div class="bg-white rounded-md mb-4">
						<div class="px-4 py-2 border-b border-gray-200">
							<h3 class="text-lg font-semibold mb-2">지원자 정보</h3>
						</div>
						<div class="flex">
							<div class="mx-2 mt-4 flex-grow max-w-40">
								<p><strong>아이디</strong></p>
								<p><strong>이름</strong></p>
								<p><strong>나이</strong></p>
								<p><strong>연락처</strong></p>
								<p><strong>주소</strong></p>
							</div>
							<div class="mx-2 mt-4">
								<p>{application.author}</p>
								<p>{application.name}</p>
								<p>{calculateAge(application.birth)}</p>
								<p>{application.phone}</p>
								<p>{application.location}</p>
							</div>
						</div>
					</div>
					<hr class="border-t border-gray-300 opacity-50" />
					<div class="flex">
						<div class="mx-2 mt-4 flex-grow max-w-40">
							<p><strong>지원서 번호</strong></p>
							<p><strong>제출일</strong></p>
							<p><strong>승인 상태</strong></p>
						</div>
						<div class="mx-2 mt-4">
							<p>{application.id}</p>
							<p>{formatDate(application.createdAt)}</p>
							<p>
								{#if application.approve === true}
									<span class="text-sm text-emerald-700">승인</span>
								{:else if application.approve === false}
									<span class="text-sm text-rose-600">미승인</span>
								{:else}
									<span class="text-sm text-orange-500">진행중</span>
								{/if}
							</p>
						</div>
					</div>
					<div class="mx-2 my-2">
						<strong>작성 내용</strong>
						<div class="p-3 bg-gray-100 rounded overflow-auto mt-1" style="max-height: 200px;">
							{application.body}
						</div>
					</div>

					{#if application.approve == null}
						<div class="text-center mt-2 flex justify-center items-center space-x-2">
							{#if application.author == rq.member.username}
								<button
									class="btn btn-active btn-primary btn-sm"
									on:click={() => goToEditPage(application.id)}>지원서 수정</button
								>
							{/if}

							{#if application.author == rq.member.username}
								<button
									class="btn btn-active btn-sm"
									on:click={() => deleteApplication(application.id)}>지원서 삭제</button
								>
							{/if}
						</div>
					{/if}
				</div>
			</div>
		</div>
	</div>
{:catch error}
	<p class="text-error">데이터를 로드하는 동안 오류가 발생했습니다: {error.message}</p>
{/await}
