# Swift에서 `bisect` 구현하기

## 발단
- Python에서 유용하게 사용한 라이브러리와 메서드를 Swift에서 구현해 보자.
- 이분 탐색은 많이 나오는 내용이다.

## 코드 (with Claude)
```Swift
func bisectLeft<T: Comparable>(_ array: [T], _ target: T) -> Int {
    var low = 0
    var high = array.count

    while low < high {
        let mid = (low + high) / 2
        if array[mid] < target {
            low = mid + 1
        } else {
            high = mid
        }
    }
    return low
}

func bisectRight<T: Comparable>(_ array: [T], _ target: T) -> Int {
    var low = 0
    var high = array.count

    while low < high {
        let mid = (low + high) / 2
        if target < array[mid] {
            high = mid
        } else {
            low = mid + 1
        }
    }
    return low
}
```

## 해설
- 먼저 상기 코드는 제네릭을 사용해서, `Comparable`한 모든 배열에 대해 작동한다.
- `bisectLeft` 함수는 주어진 값이 들어갈 수 있는 가장 왼쪽 위치(index)이다.
  - 그렇기 때문에 주어진 값보다 작은 값의 수를 셀 때도 사용할 수 있다.
- `bisectRight` 함수는 주어진 값이 들어갈 수 있는 가장 오른쪽 위치(index)이다.
  - 마찬가지 원리로 주어진 값보다 작거나 같은 값의 수를 셀 때 사용할 수 있다.
- high 부분이 `arr.count - 1`이 아니고 `arr.count`일 때의 몇 가지 이점이 있다.
  - 빈 배열을 자연스럽게 처리할 수 있다. `low == 0`, `high == 0`이라면 아예 `while`에 진입하지 않기 때문.
- 두 함수의 차이는 같은 값을 만났을 때의 처리에 있는데, 전자의 경우 왼쪽으로 이동하고 후자의 경우 오른쪽으로 이동한다.
  - `bisectLeft`에서는 `target`이 `arr[mid]`보다 작을 때만 따로 처리한다.
  - `bisectRight`에서는 `target`이 `arr[mid]`보다 클 때만 따로 처리한다.
  - 두 경우 모두 작은 값 앞에서는 `low = mid + 1`, 큰 값 앞에서는 `high = mid`를 활용한다.
  - 즉 같은 값일 때 뭘 활용해서 값을 판단하냐, 의 차이인 것!
- 정확히 같은 값이 있다는 것이 보장되고, 이것의 위치를 찾아야 한다면, `bisectLeft`를 사용하면 된다.
- 상기 메서드는 오름차순 정렬되어 있는 배열에서 사용할 수 있다.
  - 내림차순 정렬이 되어 있다면 상술한 "따로 처리하는 경우"의 부등호 방향을 바꿔 주면 된다.
  - 이 경우 `bisectLeft`는 주어진 수보다 작거나 같은 수의 갯수가 된다.
  - 그럼 `bisectRight`는 주어진 수보다 작은 수의 갯수가 되겠고.

## 활용 예시
[LeetCode No.1351 Count Negative Numbers in a Sorted Matrix](https://leetcode.com/problems/count-negative-numbers-in-a-sorted-matrix/submissions/1402963440?envType=study-plan-v2&envId=binary-search)
```Swift
class Solution {
    func countNegatives(_ grid: [[Int]]) -> Int {
        var result = 0

        for row in grid {
            let numberOfNegatives = row.count - bisectRight(arr: row)
            result += numberOfNegatives
        }

        return result
    }

    func bisectRight(arr: [Int], target: Int = 0) -> Int {
        var low = 0
        var high = arr.count

        while low < high {
            let mid = (low + high) / 2

            if arr[mid] < target {
                high = mid
            } else {
                low = mid + 1
            }
        }

        return low
    }
}
```
- `grid`의 가로 세로 길이는 100이 최대이므로, 각각의 줄을 이분 탐색하여 O(nlogn)의 시간 복잡도로 해결할 수 있다고 판단
- 내림차순 정렬 되어있는 배열에서 0보다 작은 수의 갯수를 구하는 경우
  - 상술한 부등호의 반전을 사용하고, 음수의 갯수를 구하는 것이므로 `target`에 기본값을 0으로 주고 함수 호출