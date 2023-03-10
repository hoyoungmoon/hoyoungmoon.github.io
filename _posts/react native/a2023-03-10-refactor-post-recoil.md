---
title: "[React Native] Recoil로 포스트 상태관리하기"
date: 2023-03-10
categories: react-native
---

현재 상태 (문제점)

어려운점 및 개선 방안

- 생성을 하는 건 atomFamily로 하면 된다
- 포스트 리스트와 포스트 상세보기 모두에서 포스트 상태값을 변경할 수 있는 액션들이 존재한다. 하나의 아이템이 변경된다고 리스트를 갱신할 수는 없다. 따라서 아이템별로 atomFamily를 등록해보았다.
  또 다른 문제점은 여러 곳에서 보이는 포스트의 데이터를 호출하기 위한 api 응답값이 다 제각각이다. 이건 포스트 상세보기의 api를 기준으로 모든 타입을 다 포함하는 마스터 타입을 기준으로 수정이 될때만 해당 데이터를 기준으로 다른 포스트들을 덮어쓰기로 하였다.
  예를 들어 포스트를 수정하는 단계를 다 거치고 나면 수정이 완료된 포스트의 상세보기 뷰로 이동 후 해당 마스터 데이터를 atomFamily로 저장하여 다른 포스트들도 이 데이터를 따르도록 하는 방식이다.

개선 계획

```
function usePost<T extends DeepPartial<Post>>(postNumber: number, localPostData: T): PostValueOrUpdater<T> {
	const iPostNo = Number(postNumber);
	const post = useRecoilValue(postSelector(iPostNo));
	const postUpdateAction = useSetRecoilState(postUpdate(iPostNo));
	const postDeleteAction = useSetRecoilState(postDelete(iPostNo));
	const postLikeAction = useSetRecoilState(postLikeSelector(iPostNo));
	const [_, postResetAction] = useRecoilState(postResetSelector);

	const postData = { ...localPostData, ...post };
	const actions: PostAction<T> = {
		POST_UPDATE: (updateData) => {
			console.log("POST_UPDATE", JSON.stringify(updateData, null, 2));
			if (!updateData) {
				return;
			}
			postUpdateAction(updateData);
		},
		POST_DELETE: () => {
			postDeleteAction(true);
		},
		POST_LIKE: () => {
			postLikeAction({
				iLikeCount: post.iLikeCount || localPostData?.iLikeCount || 0,
				sIsDoLike: post.sIsDoLike || localPostData?.sIsDoLike
			});
		},
		POST_RESET: () => {
			postResetAction();
		}
	};

	return useMemo(() => ({ post: postData, actions }), [localPostData, post]);
}
```

포스트 고유값을 이용하여 atomFamily로 관리한다

서로 다른 응답데이터들의 포스트들 타입 만족하기

마스터 타입을 만족하는지 타입 체크가 필요했다
T extends Partial<MasterPostType>으로 MasterPostType에 포함되는 타입을 받기로 하였다.

array, object 프로퍼티도 모두 Partial이 적용될 수 있도록 DeepPartial로 대체하였다.

```
type DeepPartial<T> = T extends Array<infer K>
	? DeepPartial<K>[]
	: T extends object
	? { [P in keyof T]?: DeepPartial<T[P]> }
	: T;
```
