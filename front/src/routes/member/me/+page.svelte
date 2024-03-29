<script lang="ts">
	import rq from '$lib/rq/rq.svelte';
	import { onMount } from 'svelte';

	function navigateToModifyPage() {
		rq.goTo('/member/social/modify');
	}

	onMount(async () => {
		try {
			await rq.initAuth(); // 로그인 상태를 초기화
			if (rq.isLogout()) {
				rq.msgError('로그인이 필요합니다.');
				rq.goTo('/member/login');
				return;
			}
		} catch (error) {
			console.error('인증 초기화 중 오류 발생:', error);
			rq.msgError('인증 과정에서 오류가 발생했습니다.');
			rq.goTo('/member/login');
		}
	});

	async function loadMyPosts() {
		const { data } = await rq.apiEndPoints().GET('/api/member/myposts', {});
		return data;
	}

	async function loadMyApplications() {
		const { data } = await rq.apiEndPoints().GET('/api/member/myapplications', {});

		return data;
	}

	async function loadMyComments() {
		const { data } = await rq.apiEndPoints().GET('/api/member/mycomments', {});
		return data;
	}

	async function loadMyInterest() {
		const { data } = await rq.apiEndPoints().GET('/api/member/myinterest', {});
		return data;
	}

	function summarizeBody(body) {
		return body.length > 10 ? `${body.slice(0, 10)}...` : body;
	}

	function goToApplicationsList(postId) {
		window.location.href = `/applications/list/${postId}`;
	}
</script>

<div class="flex justify-center items-center min-h-screen bg-base-100">
	<div class="container mx-auto px-4">
		<div class="w-full max-w-4xl mx-auto my-10">
			<div class="text-center">
				<div class="font-bold text-xl">내 정보</div>
				<div class="mt-3 text-gray-500">현재 로그인한 회원의 정보입니다.</div>
			</div>
			<div class="divider"></div>
			<div class="grid grid-colos-1 gap-4">
				<div class="flex justify-between items-center w-full">
					<div class="font-bold">아이디</div>
					<div class="text-gray-500 flex-auto text-right">{rq.member.username}</div>
				</div>
				<div class="flex justify-between items-center w-full">
					<div class="font-bold">이름</div>
					<div class="text-gray-500 flex-auto text-right">{rq.member.name}</div>
				</div>
				<div class="flex justify-between items-center w-full">
					<div class="font-bold">휴대폰 번호</div>
					<div class="text-gray-500 flex-auto text-right">{rq.member.phoneNumber}</div>
				</div>
				<div class="flex justify-between items-center w-full">
					<div class="font-bold">성별</div>
					<div class="text-gray-500 flex-auto text-right">
						{#if rq.member.gender === 'MALE'}
							남자
						{:else if rq.member.gender === 'FEMALE'}
							여자
						{/if}
					</div>
				</div>
				<div class="flex justify-between items-center w-full">
					<div class="font-bold">주소</div>
					<div class="text-gray-500 flex-auto text-right">{rq.member.location}</div>
				</div>
				<div class="flex justify-between items-center w-full">
					<div class="font-bold">생년월일</div>
					<div class="text-gray-500 flex-auto text-right">{rq.member.birth}</div>
				</div>
				<div>
					<button class="w-full btn btn-ghost" on:click={navigateToModifyPage}
						>회원 정보 수정</button
					>
				</div>
			</div>
			<div class="divider"></div>
			<div class="flex justify-center">
				<div class="join">
					<label for="my_modal_1" class="btn btn-ghost join-item">작성 공고</label>
					<input type="checkbox" id="my_modal_1" class="modal-toggle" />
					<div class="modal" role="dialog">
						<div class="modal-box">
							{#await loadMyPosts()}
								<div class="flex items-center justify-center min-h-screen">
									<span class="loading loading-dots loading-lg"></span>
								</div>
							{:then { data: posts }}
								{#each posts ?? [] as post, index}
									<a href="/job-post/{post.id}" class="card-link">
										<div class="card">
											<div class="text-sm text-gray-500">no.{index + 1}</div>
											<div class="text-lg font-bold truncate">{post.title}</div>
										</div>
									</a>
									<button
										class="btn btn-primary my-3 w-full"
										on:click={() => goToApplicationsList(post.id)}>지원서 확인</button
									>
									<div class="divider"></div>
								{/each}
							{/await}
						</div>
						<label class="modal-backdrop" for="my_modal_1">Close</label>
					</div>
					<label for="my_modal_2" class="btn btn-ghost join-item">지원 현황</label>
					<input type="checkbox" id="my_modal_2" class="modal-toggle" />
					<div class="modal" role="dialog">
						<div class="modal-box">
							{#await loadMyApplications()}
								<p>loading...</p>
							{:then { data: applicationDtoList }}
								{#each applicationDtoList ?? [] as applicationDto}
									<a href="/applications/detail/{applicationDto.id}" class="card-link">
										<div class="card">
											<div class="text-sm text-gray-500">{applicationDto.jobPostName}</div>
											<div class="text-lg font-bold truncate">
												{summarizeBody(applicationDto.body)}
											</div>
											<div class="divider"></div>
										</div>
									</a>
								{/each}
							{/await}
						</div>
						<label class="modal-backdrop" for="my_modal_2">Close</label>
					</div>
					<label for="my_modal_3" class="btn btn-ghost join-item">작성 댓글</label>
					<input type="checkbox" id="my_modal_3" class="modal-toggle" />
					<div class="modal" role="dialog">
						<div class="modal-box">
							{#await loadMyComments()}
								<p>loading...</p>
							{:then { data: commentsDtoList }}
								{#each commentsDtoList ?? [] as commentsDto}
									<a href="/job-post/{commentsDto.jobPostId}" class="card-link">
										<div class="card">
											<div class="text-sm text-gray-500">{commentsDto.jobPostId}번 공고</div>
											<div class="text-lg font-bold truncate">{commentsDto.content}</div>
											<div class="divider"></div>
										</div>
									</a>
								{/each}
							{/await}
						</div>
						<label class="modal-backdrop" for="my_modal_3">Close</label>
					</div>
					<label for="my_modal_4" class="btn btn-ghost join-item">관심 공고</label>
					<input type="checkbox" id="my_modal_4" class="modal-toggle" />
					<div class="modal" role="dialog">
						<div class="modal-box">
							{#await loadMyInterest()}
								<p>loading...</p>
							{:then { data: interestDtoList }}
								{#each interestDtoList ?? [] as interestDto}
									<a href="/job-post/{interestDto.id}" class="card-link">
										<div class="card">
											<div class="text-sm text-gray-500">{interestDto.id}번 공고</div>
											<div class="text-lg font-bold truncate">{interestDto.title}</div>
											<div class="divider"></div>
										</div>
									</a>
								{/each}
							{/await}
						</div>
						<label class="modal-backdrop" for="my_modal_4">Close</label>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>
