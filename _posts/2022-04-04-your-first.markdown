---
layout: post
title: 'js(jquery, html) 세로 줄 병합'
published: true
categories: js
---

왼쪽부터 좌측의 dept에 따라 열병합을 해야 될 일이 생겼다.
총 5가지 방법으로 구현 하였고 그 방법을 기술한다.

### 1. 첫열 병합후 비교 열은 가장 왼쪽일때만 비교

- 열병합에 있어서 제일 복잡한 부분은 병합시 병합된 메인만 'rowspan' 값이 부여되고 나머지 값은 아예 삭제 되어 다음 열을 비교시 'td' 의 위치가 지속적으로 바뀐다는 점이다.
- 해당 문제를 해결하기 위해 비교가 되는 부분의 td 값이 0 이고 해당 tr의 td의 갯수가 최대 td의 갯수보다 적을 때만 비교 하도록 하였다.

## Code
// js, jquery, html, spring

    // 테이블 id, 왼쪽부터 결합할 행 갯수(ex. 'userTable',5)
    function rowMerge2(tableName, colums){
         let setTable = $("#"+tableName+" > tbody");
         let totalColums = setTable.find('tr:eq(0)').find('td').length;
         let totalRow = setTable.find('tr').length;
         
         for(let k = 0; k < totalRow; k++){                                       // 시작 로우 부터 반복
            for(let j = 0; j < colums; j++){                                    // 컬럼 개수 만큼
               let thisArea = setTable.find('tr:eq('+ k +')').find('td:eq('+ j +')');
               //console.log(thisArea.text());         // 실제 값
               let thisTdCount = thisArea.parent().find('td').length;            
               if(j == (thisTdCount - totalColums + colums)){               
                  break;
               }
               
               // console.log(k + " " + j);         // k = 행 , 반복되는 열
               let thisTd = thisArea.closest('td').index();                        // 비교될 부분
               let thisTr = thisArea.parent().closest('tr').index();            
               let thisText = thisArea.text();
               
               for(let i = 1; i < (totalRow + 1); i++){            
                  let nextTr = thisArea.parent().parent().find('tr:eq('+(thisTr+i)+')');   //비교할 부분
                  let nextTdCount = nextTr.find('td').length;
                  let nextTdNumber = thisTd - thisTdCount + nextTr.find('td').length;
                  let nextTd = nextTr.find('td:eq('+nextTdNumber+')');
                  let nextText = nextTd.text();            
                  
                  if((nextText != thisText) || (nextTdNumber != 0)){                  // 다르거나,
                     //console.log("("+thisText+")다른 항목입니다 : " + nextText);            
                     // 비교할 영역의 상위 dept가 합쳐져있지 않은경우
                     thisArea.attr("rowspan", i);                             
                     // (비교할 값이 처음으로 오지 않은경우)
                     break;
                  }else{
                     //console.log("("+thisText+")같은 항목입니다 : " + nextText);
                     nextTd.remove();
                  }   
               }
            }                  
         }
      }


### 2. 삭제 코드 부여

- 위의 1번 방법은 이해 하기 어려울 뿐더라 설명하고 있는 나도 햇갈렸다.
- 햇갈리는 요인은 병합시 다음 행의 td의 위치가 바뀌는 부분이 어렵다고 판한, 
병합을 해도 삭제를 하지 않고 비교(td의 위치가 바뀌지 않음), 추후 삭제 코드 값을 부여하여 한번에 삭제한다.


## Code
// js, jquery, html, spring

    // 테이블 id, 왼쪽부터 결합할 행 갯수(ex. 'userTable',5)
    function rowMerge2(tableName, colums){
            let setTable = $("#"+tableName+" > tbody");                                 // 테이블 호출
            //let totalColums = setTable.find('tr:eq(0)').find('td').length;                  // 전체 열 갯수
            let totalRow = setTable.find('tr').length;                                 // 전체 행 갯수
            let sameCode = 1;
            for(let k = 0; k < totalRow; k++){                                       // 시작 로우(열) 부터 반복
               for(let j = 0; j < colums; j++){
                  let thisArea = setTable.find('tr:eq('+ k +')').find('td:eq('+ j +')');
                  let thisText = thisArea.text();
                  
                  // 삭제될 부분은 무시
                  if(thisArea.attr("class") == "deleteCode"){
                     continue;
                  }      
                  
                  // 비교 시작
                  for(let i = 1; i < (totalRow + 1 - k); i++){   
                     let nextArea = setTable.find('tr:eq('+ (k + i) +')').find('td:eq('+ j +')');
                     let nextText = nextArea.text();            
                        
                     if((nextText == thisText) && 
                     ((j == 0) || (nextArea.prev('td').attr("sameCode") == thisArea.prev('td').attr("sameCode")))){
                        //console.log(thisText + "  " + nextText);   
                        nextArea.attr("sameCode", sameCode);      // 상위 dept 병합여부 판별용
                        nextArea.attr("class", "deleteCode");      // 삭제될 부분 저장
                     }else{
                        //console.log(thisText + "  " + nextText);        
                        thisArea.attr("sameCode", sameCode);      // 다를경우 중단후 rowpan부여, sameCode 값 증가
                        thisArea.attr("rowspan", i);
                        sameCode = sameCode + 1;
                        break;
                     }               
                  }
               }
            }
            $('.deleteCode').remove();   // 삭제
            $("td").removeAttr("samecode");
         }

### 3. 기준을 행에서 열로 변경

- 2번 방식은 한 행에서 열 비교후 넘어 가기 때문에 다음 행으로 이동시 비교를 안해도 되는 값(deleteCode로 삭제될 값)을 채크 하는 부분이 필요했다
- 이를 개선 하기위해 한열을 전부 비교하고 다음 열로 넘어가도록 변경하였다(deleteCode 부여 수치 만큼 건너뛴다)


## Code
// js, jquery, html, spring

    // 테이블 id, 왼쪽부터 결합할 행 갯수(ex. 'userTable',5)
    function rowMerge3(tableName, colums){
            let setTable = $("#"+tableName+" > tbody");                // 테이블 호출
            let totalColums = setTable.find('tr:eq(0)').find('td').length;  // 전체 행 갯수
            let totalRow = setTable.find('tr').length;                        // 전체 열 갯수
            let sameCode = 1;
            if(colums > totalColums){
               colums = totalColums;
            }
            for(let j = 0; j < colums; j++){
               for(let k = 0; k < totalRow; k++){                                    
                  let thisArea = setTable.find('tr:eq('+ k +')').find('td:eq('+ j +')');
                  let thisText = thisArea.text();
                  
                  // 비교 시작
                  for(let i = 1; i < (totalRow + 1 - k); i++){   
                     let nextArea = setTable.find('tr:eq('+ (k + i) +')').find('td:eq('+ j +')');
                     let nextText = nextArea.text();            
                     
                     // 비교 대상이 같고, 상위 dept가 병합되어있는(혹은 상위 dept가 없는경우) 
                     //경우만 같은 요소로 취급
                     if((nextText == thisText) 
                     && ((j == 0) || 
                     		(nextArea.prev('td').attr("sameCode") == thisArea.prev('td').attr("sameCode")))){
                        //console.log(thisText + "  " + nextText);   
                        nextArea.attr("sameCode", sameCode);      	// 상위 dept 병합여부 판별용
                        nextArea.attr("class", "deleteCode");      	// 삭제될 부분 저장
                     }else{
                        //console.log(thisText + "  " + nextText);
                        thisArea.attr("sameCode", sameCode);      	// 다를경우 중단후 rowpan부여, sameCode 값 증가
                        thisArea.attr("rowspan", i);
                        k = k + i - 1;
                        sameCode = sameCode + 1;
                        break;
                     }               
                  }
               }
            }
            $('.deleteCode').remove();   // 삭제
            $("td").removeAttr("samecode");
         }

### 4. 간소화

-- 3번 방법은 맘에 들었지만, 코드가 길다고 생각 하였다. 굳이 필요 없는 부분은 다 줄이고 코드를 간략화 시켰다.
-- 다만 여러 불안정한 요소들을 채크 하는 부분을 줄였기 때문에 3번에 비해서는 에러가 나올 가능성이 크다.

## Code
// js, jquery, html, spring

    // 테이블 id, 왼쪽부터 결합할 행 갯수(ex. 'userTable',5)
    function rowMerge4(tableName, colums){
            let setTable = $("#"+tableName+" > tbody");
            let sameCode = 1;
            let k = 0; let i = 1; let j = 0;      
            while(true){
               let thisArea = setTable.find('tr:eq('+ k +')').find('td:eq('+ j +')');
               let nextArea = setTable.find('tr:eq('+ (k + i) +')').find('td:eq('+ j +')');
               if((nextArea.text() == thisArea.text()) && 
               	((j == 0) || (nextArea.prev('td').attr("sameCode") == 
                	thisArea.prev('td').attr("sameCode")))){
                  nextArea.attr("sameCode", sameCode);      			// 상위 dept 병합여부 판별용
                  nextArea.attr("class", "deleteCode");      			// 삭제될 부분 저장
                  i++;
               }else{
                  thisArea.attr("sameCode", sameCode);      			// 다를경우 중단후 rowpan부여, sameCode 값 증가
                  thisArea.attr("rowspan", i);
                  k = k + i;                        				// 삭제될 부분 넘어가기
                  if(k >= (setTable.find('tr').length-1)){
                     k = 0;
                     j++;
                  }
                  if(j >= colums)
                     break;               
                  i = 1;                              				// 비교될 부분 다음 값을 초기화
                  sameCode = sameCode + 1;
               }               
            }               
            $('.deleteCode').remove();   					// 삭제
            $("td").removeAttr("samecode");
         }

### 5. 열 선택 기능
- 실제 환경에서는 이와 같이 왼쪽에서 부터만 순차적으로 결합이 된다는 보장이 없었다.
- 유지보수를 위해 받아오는 부분을 배열과 숫자로 나누었다.
- 배열일경우 배열의 순차적으로 병합, 숫자일경우 왼쪽에서 부터 병합되도록 바꾸었다

## Code
// js, jquery, html, spring

    // 테이블 id, 왼쪽부터 결합할 행 갯수 혹은 배열(ex. ('userTable',5), ('userTable',[1,0,3,4,5]))
    // 딘 배열일 경우 가장 왼쪽을 1이 아닌 0을 기준으로 입력
    function rowMergeList(tableName, colum){
              let colums = [];
              if(typeof(colum) == 'number'){
                  for(let i = 0; i<colum; i++){
                      colums.push(i);
                  }
              }else{
                  colums = colum;
              }
             let setTable = $("#"+tableName+" > tbody");		// 테이블 호출
             let totalRow = setTable.find('tr').length;		// 전체 열 갯수
             let sameCode = 1;				// 상위 dept 병합 여부 판별 번호			
             let PrevTd = 0;					// 이전 Td 위치
             for(const j of colums){
                for(let k = 0; k < totalRow; k++){ 
                   let thisTr = setTable.find('tr:eq('+ k +')')		// 비교될 값
                   let thisArea = thisTr.find('td:eq('+ j +')');
                   let thisText = thisArea.text();                   
                   // 비교 시작
                   for(let i = 1; i < (totalRow + 1 - k); i++){ 
                	  let nextTr = setTable.find('tr:eq('+ (k + i) +')');	// 비교할 값
                      let nextArea = nextTr.find('td:eq('+ j +')');
                      let nextText = nextArea.text();            
                      
                      // 비교 대상이 같고, 
                      	상위 dept가 병합되어있는(혹은 상위 dept가 없는경우) 경우만 같은 요소로 취급
                      if((nextText == thisText) 
                    		&& ((j == colums[0]) 
                    			|| (nextTr.find('td:eq('+ PrevTd +')').attr("sameCode") == thisTr.find('td:eq('+ PrevTd +')').attr("sameCode")))){
                         //console.log(thisText + "  " + nextText);   
                         nextArea.attr("sameCode", sameCode);      // 상위 dept 병합여부 판별용
                         nextArea.attr("class", "deleteCode");      // 삭제될 부분 저장
                      }else{
                         //console.log(thisText + "  " + nextText);
                         thisArea.attr("sameCode", sameCode);      // 다를경우 중단후 rowpan부여, sameCode 값 증가
                         thisArea.attr("rowspan", i);
                         k = k + i - 1;
                         sameCode = sameCode + 1;
                         break;
                      }               
                   }
                }
                PrevTd = j;
             }
             $('.deleteCode').remove();   // 삭제
             $("td").removeAttr("samecode");
          }
