> [!abstract] 개요
> [llama3.np](https://github.com/likejazz/llama3.np) 코드를 참고하여 토큰 입력부터 출력까지의 operation flow 를 정리한 문서

# Operations

## Make Encoder Input

텍스트 입력을 받아서 아래와 같은 절차를 통해 최종 인코더의 입력으로 들어갈 input 이 만들어진다.
- **Text**
    - Raw text input. (shape: `(1, token_lengh)`)
- **Tokenizing**
    - Splitting text into tokens. (shape: `(1, token_lengh)`)
- **Token Encoding**
    - Converting tokens to numerical IDs. (shape: `(1, token_lengh)`)
- **Token Embedding**
    - Mapping token IDs to dense vectors. (shape: `(1, token_lengh, dim)`)
- **Positional Embedding**
    - Adding positional information to embeddings. (shape: `(1, token_lengh, dim)`)
- **Final Representation**    
    - Combining token embeddings with positional embeddings.

>[!notice] Notation 정리
> `l` : token 의 길이
> `d`(dim): 하나의 token 이 가지는 vector 의 크기
> `vs`(vocab size): vocab 크기
> `h` (head): attention 에서 head 의 갯수

![[Screenshot 2024-06-16 at 6.36.08 PM.png]]
![[Screenshot 2024-06-16 at 6.36.24 PM.png]]

# References

