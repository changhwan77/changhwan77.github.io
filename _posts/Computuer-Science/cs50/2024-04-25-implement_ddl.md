---
layout: single
title: C언어 - Double Linked List 포인터 관련 에러 
toc: true
date: 2024-04-25
categories:
  - CS50
tag: [C언어, 자료구조]
---

# C언어로 Double Linked List 구현하며 만날 수 있는 에러

이번 포스팅은 c언어로 이중 연결 리스트를 구현하며 만날 수 있는 에러에 대한 내용입니다. 


## 포인터 변수를 넘겨줬는데 실제 local 변수의 값이 변경되지 않는 경우

```c
node *head = NULL;
node *tail = NULL;
createDoubleLinkedList(head, tail, 3);
 
void createDoubleLinkedList(node *head, node *tail, int number)
{
    // number만큼의 Loop를 돈다.
    for (int i = 1; i <= number; i++)
    {
        // 새로운 node를 생성한다
        node *new_node = createNode(i);

        if (head == NULL)
        {
            head = new_node;
        }
        else 
        {
            while (head->next != NULL)
            {
                head = head->next;
            }
            head->next = new_node;
            new_node->previous = head;
            new_node->next = NULL;
            tail = new_node;
        }
    }
    return (node *)head;
}
 
 ```
 
위 코드를 실행하면 head와 tail 포인터에는 어떤 일이 벌어질까?

내가 기대한 건 당연히 head -> node1 -> node2 -> node3 -> tail으로 붙은 head 포인터를 예상했지만 결과는 NULL이었다. 

이유는 head 포인터에 대한 복사 값이 전달되어 실제 메모리 주소의 값이 변경되지 않았기 때문이다. 

그래서 head 포인터에 대한 주소(&head)를 넘기려면 head 포인터에 대한 포인터(**head)를 함수의 argument로 받아야 한다. 

## head 포인터를 초기화할 때 NULL과 malloc의 차이.


1. NULL로 초기화한 포인터 변수 head를 그대로 넘겨주고 그 값의 주소를 *tmp로 받으려고 했더니 에러 발생. 

왜냐하면 포인터 변수에 대한 포인터 변수이므로 tmp는 포인터에 대한 포인터 변수 구조체가 되어야 함.

하지만 이 경우에도 또 에러가 발생함.

2. 현재 작동이 되는 방식은 포인터에 대한 포인터 변수(node **head)로 인자를 받고 직접 해당 주소에 접근해서 값을 바꾸는 방식을 취하고 있음.

3. 만약에 그냥 포인터 변수 head를 넘겨받고 해당 head에 malloc을 한다면?
실제 리턴 값은 createDDL의 지역 변수가 나가서 실제 head값에 변경이 일어나지 않음. 



### 1. malloc으로 포인터를 초기화하고 함수를 통해 DDL을 만들 경우

- 함수 내에서 데이터를 조작하면 실제 주소에 있는 값이 바뀜.
- 이 때 이유는 malloc으로 main 함수 내에서 heap 메모리 영역에 실제 메모리 Location을 할당했기 때문에 해당 주소에 접근하여 실제 값을 변경하게 되기 때문이다. 

![](https://velog.velcdn.com/images/changhwan77/post/717d2cef-06d9-452a-abcb-505bea3faf65/image.png)

- 대신 head에 노드라는 구조체가 진짜로 생성되며 데이터가 0으로 초기화되고 디버깅 시 0, 1, 2, 3이 찍힌다.
    ![](https://velog.velcdn.com/images/changhwan77/post/43807db8-66c6-4280-9835-16d499b0df58/image.png)

### 2. NULL로 초기화 시키고 넘기는 경우

![](https://velog.velcdn.com/images/changhwan77/post/fa465348-e701-4cf7-9697-6d78c8aa67bc/image.png)

head라는 포인터는 data가 1인 노드를 가리켜서 바로 접근하는 것을 볼 수 있다. 

- node1 주소 

![](https://velog.velcdn.com/images/changhwan77/post/8251f9e4-5b86-4dd0-b17a-7686fc0d3d99/image.png)

- head 주소

![](https://velog.velcdn.com/images/changhwan77/post/e7404873-7818-4287-9bb7-672db787adda/image.png)

이렇게 했을 때 data를 찍어보면 어떻게 될까?

![](https://velog.velcdn.com/images/changhwan77/post/6f0e564e-cf94-4237-9e6c-ead9954cd9ea/image.png)

해당 포인터 변수에는 노드 1의 주소를 담고 있기 때문에 접근하게 되면 바로 node 1로 이동하게 되어 노드 1의 숫자인 1을 출력하게 된다. 

그래서 궁금했다. '진짜' 포인터의 개념은 뭘까?

> 포인터의 정의
![](https://velog.velcdn.com/images/changhwan77/post/202be1ed-d119-49e1-9c45-eb36cd2f623a/image.png)
포인터는 C 언어의 변수 또는 memory location에 대한 주소를 저장할 수 있는 데이터 type

내가 원하는 결과는 당연히 함수로 head를 전달해 double linked list를 생성하는 것이었고 위의 문제를 해결하기 위한 방법으로는 3가지가 있었다. 


CTO님께 물어본 결과 가비지 data를 생성하는 경우에(의미없는 데이터)는 사용하지 말라고 하심. 

그리고 node의 head의 경우에는 의미는 통하나 누가봐도 직관적이게 구조체를 설정하는 것이 중요하다며 list라는 구조체를 만들어서 


### 해결 방법

### 기존에 생각한 3가지 방법

1. head 포인터를 함수에 전달해 조작한 후 head 포인터를 반환해 할당하는 법 

```c
node* createDoubleLinkedList(node *head, node *tail, int number)
{
    // number만큼의 Loop를 돈다.
    for (int i = 1; i <= number; i++)
    {
        // 새로운 node를 생성한다
        node *new_node = createNode(i);

        if (head == NULL)
        {
            head = malloc(sizeof(node));
            tail = malloc(sizeof(node));
            head = new_node;
            tail = new_node;
        } 
        else 
        {
            node *tmp = head;
            while (tmp->next != NULL)
            {
                tmp = tmp->next;
            }
            tmp->next = new_node;
            new_node->previous = tmp;

            tail = tmp->next;
        }
    }
    return head;
}
```

변경된 head 포인터를 반환해서

```c
node *head = NULL;
head = (node *)createDoubleLinkedList(head, tail, 3);
```

미리 선언해놨던 head에 할당하면 

![](https://velog.velcdn.com/images/changhwan77/post/457f9406-6758-44fb-8dce-ad7d6ca26655/image.png)

디버깅해보면 위 이미지와 같이 createDDL에서 반환된 head 포인터가 할당된 것을 확인할 수 있다. 

2. head 포인터에 대한 포인터 변수를 인자로 받아 해당 **head의 주소로 접근(head 포인터)에 직접 연산을 통해 값을 바꾸는 방법

3. main 함수 내에서 head 포인터에 malloc을 통해 node 구조체를 만들게 되는 것. 

```c
int main(void)
{
  node *head = malloc(sizeof(node));
  node *tail = malloc(sizeof(node));
  createDoubleLinkedList(head, tail, 3);
}
```

근데 이 경우에는 head라는 포인터가 가리키는 node가 있고 이는 엄밀히는 garbage값을 node를 생성하고 그 node를 가리키는 포인터 이름을 head라고 했으므로 head가 포인터 역할을 하는 게 맞나? 라는 의문이 들었다. 

### 새로운 구조체를 통해 해결하는 방법

CTO님께 여쭤봤을 때 배운 것은, 나중에 돌아와서도 내가 직관적으로 잘 기억할 수 있는 네이밍인가? 였다. 

그러한 맥락에서 내가 제시한 방법 3가지는 그렇지 않았다고 생각했고 새로운 아이디어인 

>
list라는 구조체를 만들어 관리!

하는 방법을 알게 됐다. 

```c
// LinkedList 라는 구조체 선언
typedef struct
{
  node *head;
  node *tail;
  int size;
}
LinkedList;
```

즉, node라는 데이터 타입을 가리키는 포인터인 head, tail과 해당 리스트의 길이를 관리하는 size를 필드로 갖는 LinkedList 구조체를 만들었다. 

이를 다음과 같이 초기화한 후 LinkedList라는 데이터 타입을 가리키는 포인터 변수 list를 인자로 받고 이에 실제 LinkedList 구조체를 가지고 있는 변수의 주소 &list를 할당하여 함수 내부에서 조작되는 것들은 실제 list의 주소로 접근해 LinkedList 구조체를 수정을 가하게 되고 이는 createDDL 함수가 전부 실행이 되면 실제 값이 변경된다.

```c
int main(void) 
{
  LinkedList list;
  list.head = NULL;
  list.tail = NULL;
  list.size = 0;
  
  createDoubleLinkedList(&list, 3);
  
void createDoubleLinkedList(LinkedList *list, int number)
{
    // number만큼의 Loop를 돈다.
    for (int i = 1; i <= number; i++)
    {
        // 새로운 node를 생성한다
        node *new_node = createNode(i);

        if (list->head == NULL)
        {
            list->head = new_node;
            list->tail = new_node;
            list->size++;
        }
        else
        {
            node *tmp = list->head;
            while (tmp->next != NULL)
            {
                tmp = tmp->next;
            }
            tmp->next = new_node;
            tmp->next->previous = tmp;
            list->tail = tmp->next;
            list->size++;
        }
    }
  }
}
```