## Layout Thrashing

<img src="https://user-images.githubusercontent.com/62097867/212122109-3fac605d-2d53-42ad-a904-c62de7bee9ea.png" width="800px" />

일반적으로 스타일을 계산하고 시각적 요소가 그려질 좌표를 계산하는 과정(reflow)은 프레임당 한번씩 실행된다.

하지만 JavaScript 코드에 의해 일반적인 화면 업데이트 주기(약 16.6ms)를 벗어나 강제로 reflow가 수행되는 경우가 있다.

강제된 reflow가 한 프레임 안에서 연속적으로 여러 번 발생할 경우 긴 작업을 생성하고 프레임률이 저하되는 현상이 발생하는데 이를 **layout thrashing**이라 한다.

```javascript
element.classList.add("width-adjust"); // 1. write (invalidate Layout Tree)
element.getBoundingClientRect(); // 2. read (force a synchronous reflow) -> This can be SLOW!
```

### 1. invalidate layout tree

<img src="https://user-images.githubusercontent.com/62097867/212122212-ad59ceab-724f-4a34-811b-b2ade0aa9f54.png" width="800px" />

DOM 요소에 write 작업이 발생하면 layout tree의 일부가 dirty한 노드로 지정된다. (invalidate)
이것은 스타일과 좌표 정보가 더이상 정확하지 않고 재계산이 필요하다는 의미이다.

브라우저는 가능한 reflow 비용을 최소화하려 하기 때문에 reflow를 지연시켜 배치 처리한다.

### 2. force reflow

<img src="https://user-images.githubusercontent.com/62097867/212122353-4bb1dbcf-a126-4655-a899-60ea1b24cfcf.png" width="800px" />

하지만 현재 프레임이 완료되기 전에 invalidate한 DOM 요소에 특정한 operation을 수행하려 하면 synchronous reflow가 발생한다.

(요청한 요소의 스타일과 좌표가 계산되지 않은 상태라면 이를 즉시 계산해야하기 때문)

> 🔎 [reflow를 강제하는 브라우저 API](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)

synchronous reflow가 완료되면 브라우저는 렌더트리에 계산된 정보를 다시 캐시하고 다른 업데이트가 없다면 asynchronous reflow가 감소하거나 생략될 수 있다.

forced reflow가 항상 성능 이슈가 있는 것은 아니며 단순히 reflow 시점을 이동시키는 데에 사용할 수 있지만
synchronous reflow가 프레임 내에서 여러 번 발생한다면 layout thrashing이 발생할 수 있고 invalidation 범위에 따라 심각한 성능 이슈가 발생할 수 있으므로 가능한 피하는 것이 좋다.

## Prevent Layout Thrashing

### 1. Batch Reads and Writes

```javascript
const elements = [...document.querySelectorAll(".some-class")];

// Do all reads
const rects = elements.map((element) => element.getBoundingClientRect());

// Do all writes
elements.forEach((element) => element.classList.add("width-adjust"));

// Done! Asynchronous reflow will compute positions later.
```

- read를 일괄 처리한 후 write

- read 작업들을 캐시된 render tree에서 빠르게 수행된다.

- write 작업들은 dirty로 지정되고 asynchronous reflow동안 한번에 처리된다.

### 2. requestAnimationFrame

setTimeout과 유사하게 다음 프레임에서 실행될 함수를 예약하는 API이다.

write 작업을 다음 프레임에서 함께 실행되도록 예약하는 데 사용할 수 있다.

### 3. read-write cycle 주의

```javascript
/** bad */

function resizeAllParagraphsToMatchBlockWidth() {
  // Puts the browser into a read-write-read-write cycle.
  for (var i = 0; i < paragraphs.length; i++) {
    paragraphs[i].style.width = box.offsetWidth + "px";
  }
}
```

```javascript
/** good */

// Read.
var width = box.offsetWidth;

function resizeAllParagraphsToMatchBlockWidth() {
  for (var i = 0; i < paragraphs.length; i++) {
    // Now write.
    paragraphs[i].style.width = width + "px";
  }
}
```

https://www.webperf.tips/tip/layout-thrashing/

https://web.dev/avoid-large-complex-layouts-and-layout-thrashing/#avoid-forced-synchronous-layouts

https://gist.github.com/faressoft/36cdd64faae21ed22948b458e6bf04d5
