# 순열과 조합
순열과 조합을 이용하여 특정 array에서 데이터를 뽑는 경우를 저장하는 알고리즘 정리
  
## 순열(Permutation)
  
``` java
static void permutation(int[] arr, int[] output, boolean[] visited, int depth, int n, int r) {
		if (depth == r) {
			int[] tmp = output.clone();
			permutationOutput.add(tmp);
			return;
		}

		for (int i = 0; i < n; i++) {
			if (!visited[i]) {
				visited[i] = true;
				output[depth] = arr[i];
				permutation(arr, output, visited, depth + 1, n, r);
				visited[i] = false;
			}
		}
	}
```

## 조합(Combination)
  
  ``` java
static void combination(int[] arr, int[] output, int index, int depth, int n, int r) {
		if (r == 0) {
			int[] tmp = output.clone();
			combinationOutput.add(tmp);
			return;
		}

		for (int i = depth; i < n; i++) {
			output[index] = arr[i];
			combination(arr, output, index + 1, i + 1, n, r - 1);
		}
	}
```
  
nCr = n-1Cr-1 + n-1Cr 이용한 방법
  
``` java
static void combination2(int[] arr, int[] output, int index, int target, int n, int r) {
		if (r == 0) {
			int[] tmp = output.clone();
			combinationOutput.add(tmp);
			return;
		}

		if (target == n)
			return;

		output[index] = arr[target];
		combination2(arr, output, index + 1, target + 1, n, r - 1);
		combination2(arr, output, index, target + 1, n, r);
	}
```
