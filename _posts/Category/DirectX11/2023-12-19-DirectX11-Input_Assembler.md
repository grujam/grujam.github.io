---
layout: single
title: "DirectX 11 - Input Assembler"
categories: DirectX11
classes: wide
---

### DirectX 11 - Input Assembler

Input Assembler는 렌더링 파이프라인 중 첫 번째 과정으로 버퍼에서 점, 선, 삼각형 등 기본이 되는 데이터를 읽고 정리하며 다음 단계(Vertex Shader Stage)로 넘겨주는 작업을 진행합니다.

**Input Assembler 적용**

Input Assembler 단계는 5가지 절차를 통해 진행됩니다.


**1. Input Buffer**

IA에 적용되는 Input Buffer는 Vertex Buffer와 Index Buffer 두가지가 존재합니다. Vertex Buffer는 정점 데이터를 저장하며, Index Buffer는 정점들을 잇는 정보를 저장합니다.

그래픽을 표현하는데에 필요한 정점 데이터를 저장하는 Vertex Buffer는 최소 한개 이상 존재해야하며, Index Buffer는 어떤 그래픽을 표현하느냐에 따라 선택적으로 적용할 수 있습니다.

Shader에서 Buffer를 필요하지 않는 경우, Buffer를 설정하고 Binding하는 작업은 불 필요 합니다.

```cpp
// 삼각형의 세 정점 버퍼를 만드는 과정

//Vertex는 Position을 저장하는 구조체
Vertex vertices[3];

vertices[0].Position = Vector3(0.0f, 0.0f, 0.0f);
vertices[1].Position = Vector3(0.0f, 0.5f, 0.0f);
vertices[2].Position = Vector3(0.5f, 0.0f, 0.0f);

//Vertex Buffer 생성
D3D11_BUFFER_DESC desc;

//항상 메모리를 초기화하여 쓸데 없는 메모리가 들어가는 것을 방지한다.
ZeroMemory(&desc, sizeof(D3D11_BUFFER_DESC));

//버퍼의 들어가는 바이트의 크기 = vertices 배열의 크기
desc.ByteWidth = sizeof(vertices);

// Vertex Buffer로 사용함을 알려준다
desc.BindFlags = D3D11_BIND_VERTEX_BUFFER;

//들어가는 데이터의 주소를 저장, vertices
D3D11_SUBRESOURCE_DATA subResource = { 0 };
subResource.pSysMem = vertices;

//Buffer를 desc와 subResoruce, vertexBuffer 3가지를 통해 Device로 부터 생성하고 HRESULT값이 fail아닌지 검사한다.
Check(D3D::GetDevice()->CreateBuffer(&desc, &subResource, &vertexBuffer));
```


**2. Input Layout Object**

Input Layout Object는 입력 상태에 대한 캡슐화를 통해 IA 단계에서 바인딩 된 데이터에 대한 설명을 셰이더에게 제공합니다.

*ID3D11Device::CreateInputLayout*을 통해 생성되며, 생성 시 D3D11_INPUT_ELEMENT_DESC 구조체를 통해 데이터에 대한 설명과 3번째 매개변수를 통해 컴파일 된 셰이더에 대한 포인터를 제공받습니다.

D3D11_INPUT_ELEMENT_DESC는 마이크로소프트에서 제공하는 자료를 보면 아래와 같은 형태를 갖습니다.

![](/assets/images/DirectX/INPUTELEMENTDESC.PNG)

해당 구조체를 통해 셰이더에서 사용될 Semantic 이름과 번호, 변수 타입과 각 바이트가 사용될 범위 등을 설정 할 수 있습니다.


**3. Bind Objects to IA Stage**

세 번째로는 생성된 버퍼들과 오브젝트를 IA 단계에 바인딩 하는 것입니다.

두 바인딩은 *ID3D11DeviceContext::IASetVertexBuffers*와 *ID3D11DeviceContext::IASetInputLayout*을 통해 진행할 수 있습니다.

InputLayout은 생성된 오브젝트를 바로 입력하여 바인딩 할 수 있지만, VertexBuffer의 경우 5개의 파라미터를 통해 입력을 받습니다.

IASetVertexBuffers를 통해 들어가는 데이터는 다음과 같습니다.   
1. StartSlot: 버퍼가 IA에 입력될 때 들어가는 슬롯 번호   
2. NumBuffers: 들어가는 버퍼의 갯수   
3. ppVertexBuffers: 생성된 버퍼   
4. pStrides: 각 데이터의 크기
5. pOffset: 데이터 시작 번호


```cpp
//생성된 3개의 점 데이터를 보유한 VertexBuffer를 바인딩하는 함수
UINT stride = sizeof(Vertex);
UINT offset = 0;

ID3D11DeviceContext::IASetVertexBuffers(0, 1, &vertexBuffer, &stride, &offset);
```

해당 함수를 통해 0번 슬롯을 통해 들어가는 1개의 vertexBuffer가 바인딩되며, 각 데이터의 크기는 sizeof(Vertex)이며 offset부터 읽기 시작하라는 것을 의미합니다.


**4. Specify the Primitive Type**

입력된 정점 데이터들을 어떤 Primitive Topology에 따라 이을 것인지 정하는 단계입니다.

*ID3D11DeviceContext::IASetPrimitiveTopology*를 통해 적용하며, 삼각형을 그린다고 가정하면 내부를 칠하거나 선으로만 연결하는 등의 방식을 정할 수 있습니다.


**5. Call Draw Methods**

마지막은 입력된 데이터를 통한 Draw를 콜해주는 것입니다.

Draw 명령은 셰이더에게 어떤 technique과 pass를 사용하여 몇개를 그릴 것인지 정한다면, 셰이더가 이를 파악하여 사용할 vs rs ps등을 지정한 후 렌더링합니다.

*참조: <https://learn.microsoft.com/en-us/windows/win32/direct3d11/d3d10-graphics-programming-guide-input-assembler-stage-getting-started>*
