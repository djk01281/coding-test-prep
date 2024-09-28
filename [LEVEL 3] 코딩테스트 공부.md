# [LEVEL 3] 코딩테스트 공부

## TLDR
> [!IMPORTANT]
> 1. 2D DP 시, 값을 `clamp`해줄 수 있겠다. 즉, 범위에 들게 만드는 문제를 한 점으로 고정시키는 방식.
> 2. `matrix` 생성 시, `row`와 `col` 길이 validate하기. `0`이 아닌지 확인하기.

<br>

**문제 링크** : [코딩 테스트 공부](https://school.programmers.co.kr/learn/courses/30/lessons/118668?language=javascript) <br>

**채점 결과**

[1차]

정확성: 50.0 <br>
효율성: 45.0 <br>
합계: 95.0 / 100.0 <br>

[2차]

정확성: 50.0 <br>
효율성: 50.0 <br>
합계: 100.0 / 100.0 <br>

## Thought Process

### Goal

**unbounded knapsack.**

`target`을 **넘게** 만드는 **Min Cost** 구하기.

<br>

### Decision Space
![image](https://github.com/user-attachments/assets/d9c467ba-d521-46a5-9f05-284c6c2f4c53)

> 케이스가 너무 많다고 느낌.

즉, Naive한 방식은 탈락.

<br>

### Min Cost Calculation
경우의 수가 아닌 Min Cost를 구하면 되니까 `Greedy`와 `DP` 가능.

1. `Greedy` : 최적의 선택하려면 미래의 일을 미리 알아야 됨. **탈락.**
2. `DP` : 변수도 2개고, 역행하지도 않음. **선택.**

<br>

### 2D DP

![image](https://github.com/user-attachments/assets/7f3b5b76-190d-4e76-b118-aea79f3b1cfc)

**목적** 

빨간색 영역이 `target`. Min Cost 구하기.

<br>

**문제점**

빨간색 영역이 고정되어 있지 않기 때문에 (제한사항으로 말고는), 노란색 영역의 크기를 미리 알 수 없음.
즉, 제한 사항에 맞춰서 `matrix` 크기를 정해주지 않는 이상 알 수 없음.
다만, 효율성 테스트가 있기 때문에 사실상 제한 사항은 없는 셈.

<br>

### Simplified Problem

![image](https://github.com/user-attachments/assets/0b1e8fd0-7a8b-4540-b57e-513474ae2ab2)

**편한 문제**

`target`을 넘는 -> `target`을 **맞추는** Min Cost 구하는 Unbounded Knapsack

<br>

**편하게 만드는 방법**

`Math.min(max_cop, cop)` 형태로 강제로 `clamp` 해주면 될 거라 판단했음.

즉, 범위를 넘어가도 실제로는 넘어가지 않은 것처럼.

<br>

> 이 판단을 못했어도, 제한 사항에 맞춘 크기로라도 만들었을 듯.

<br>

### Redefined Problem

2D DP. 마지막 칸 값을 `return`.

1. DP 돌면서 : `for` loop 2개.
2. 이동 가능한 방식 구하기 : 그냥 하나씩 비교.
3. 최소값 구하기 : 단순 비교.

<br>

### Edge Cases
초기값이 `target`보다 클 때, 조금 불안해보였던 것 같음.

<br>


## Submition

```js
function traverse2DArray(array, callback){
    let rows = array.length
    let cols = array[0].length
    
    for(let i = 0; i < rows; i++){
        for(let j = 0; j < cols; j++){
            callback(i, j)
        }
    }
}

function solution(alp, cop, problems) {
    const maxAlp =  Math.max(...problems.map((problem)=>problem[0]))
    const maxCop = Math.max(...problems.map((problem)=>problem[1]))
    
    if(alp > maxAlp && cop > maxCop){
        return 0
    }
    
    const rows = Math.max(0, maxAlp - alp) + 1
    const cols = Math.max(0, maxCop - cop) + 1
        
    const dp = Array(rows).fill().map(() => Array(cols).fill(0));
        
    const getAvailableProblems = (currentAlp, currentCop) => problems.filter((problem)=>{
        const [alp_req, cop_req, alp_rwd, cop_rwd, cost] = problem
        return (currentAlp >= alp_req && currentCop >= cop_req)
    })
    
    const trimIndex = (i, j) =>  [Math.min(i, rows-1), Math.min(j, cols-1)]

    
    traverse2DArray(dp, (i, j) => {
        if(i < rows-1){
            dp[i+1][j] = dp[i+1][j] === 0 ? dp[i][j] + 1 : Math.min(dp[i+1][j], dp[i][j] + 1)
        }
        
        if(j < cols-1){
            dp[i][j+1] = dp[i][j+1] === 0 ? dp[i][j] + 1 : Math.min(dp[i][j+1], dp[i][j] + 1)
        }
        
        getAvailableProblems(i+alp, j+cop).forEach((problem) => {
            const [alp_req, cop_req, alp_rwd, cop_rwd, cost] = problem
            const [nextI, nextJ] = trimIndex(i + alp_rwd, j + cop_rwd)
            
            dp[nextI][nextJ] = dp[nextI][nextJ] === 0 ? dp[i][j] + cost : Math.min(dp[nextI][nextJ], dp[i][j] + cost)
        })
    })
    
    return dp[rows-1][cols-1];
}
```

<br>

## Possible Improvements

1. 크게 문제될 건 없어 보이는데, [효율성] 3번 테스트 케이스 `TLE`. 진짜 모르겠넹 😢

> [효율성] 3번 테스트 케이스만 0되게 찍었는데 "정답입니다" 이러네.
> grid가 많은데 답이 `0`인 경우가 있을 수 있나...?

<br>

2. 첫번째 칸이 `(0, 0)`이 아닌 **초기값**을 나타내기 때문에, 실수할 여지가 있을 듯. `index` 처리도 추상화해줬으면 더 편했을 것 같다.

특히 `getAvailableProblems(i+alp, j+cop)` 이 부분. 까먹었으면 어떡해.

```js
// before
getAvailableProblems(i+alp, j+cop)

//after
const [currentAlp, currentCop] = [i+alp, j+cop]
getAvailableProblems(currentAlp, currentCop)
```

> 최소한 이렇게라도 해주자..

<br>

3. ternary operator 부분 너무 복잡함. 복잡하면 검산 안할 거잖아. 애초에 `0`이 아니라, `Infinity`로 초기값 넣어줬으면 `Math.min()`으로 바로 넣을 수 있었을 듯. `visited` 했는지 알 필요가 있는 문제도 아니고.

```js
// before
const dp = Array(rows).fill().map(() => Array(cols).fill(0));
...
dp[nextI][nextJ] = dp[nextI][nextJ] === 0 ? dp[i][j] + cost : Math.min(dp[nextI][nextJ], dp[i][j] + cost)

//after
const dp = Array(rows).fill().map(() => Array(cols).fill(Infinity));
...
const [currentAlp, currentCop] = [i+alp, j+cop]
dp[nextI][nextJ] = Math.min(dp[nextI][nextJ], dp[i][j] + cost)
```
