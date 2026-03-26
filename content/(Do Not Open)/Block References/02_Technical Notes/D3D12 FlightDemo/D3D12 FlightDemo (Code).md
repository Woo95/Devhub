# Code Block References
---

## Definitions (Kor)
```
상수 버퍼 뷰(Constant Buffer View):
	쉐이더가 변환 행렬, 조명 파라미터, 카메라 위치 같은 상수 uniform 데이터를 읽을 수 있도록 GPU가 접근할 수 있게 해주는 뷰.
```

^12909b

```
셰이더 리소스 뷰(Shader Resource View):
	쉐이더가 렌더링 중에 텍스처나 구조화된 버퍼 같은 다양한 GPU 리소스에 접근할 수 있게 해주는 설명자(descriptor). 
```

^4b061d

```
무순서 접근 뷰(Unordered Access View):
	쉐이더나 리소스(텍스처나 버퍼 등)를 임의의 순서로 읽고 쓸 수 있게 하는 설명자(descriptor). CBV나 SRV처럼 제약이 없고, 읽기/쓰기가 가능.
```

^3651ae

```
샘플러(Sampler): 
	텍스처를 샘플링할 때 사용하는 필터링 방식이나 주소 모드(랩, 클램프 등)를 정의하는 설명자(descriptor).
```

^be5c68

```
고급 셰이더 언어(High-Level Shader Language):
	Microsoft에서 개발한 쉐이더 프로그래밍 언어로, Direct3D 렌더링 파이프라인에서 GPU 프로그램을 작성할 때 사용.
```

^58ccc4

```
정점 셰이더(Vertex Shader):
	그래픽스 파이프라인에서 정점 데이터를 (위치, 법선 등) 모델 공간에서 클립 공간으로 변환하는 프로그래밍이 가능한 단계.
```

^cfc0fb

```
픽셀 셰이더(Pixel Shader):
	각 픽셀의 최종 색상을 결정하는 프로그래밍이 가능한 단계. 주로 텍스처 데이터나 조명 계산을 할때 사용.
```

^c1153e

```
상수 버퍼(Constant Buffer):
	행렬이나 조명 같은 프레임 단위의 동적 데이터를 저장하는 버퍼. CBV를 통해 접근.
```

^fe87af

```
명령 할당자(Command Allocator):
	한 프레임 동안 GPU 명령을 기록할 때 필요한 메모리를 관리하는 객체.
```

^ebaf60

---
## Code

### Setup (Kor)
```cpp
bool Game::Initialize()
{
    // 1. 디바이스 초기화
    if (!D3DApp::Initialize()) 
	    return false;   
        
	// 2. 그래픽스 파이프라인 설정
	BuildRootSignature();
	BuildShadersAndInputLayout();
	BuildPSOs();

	// 3. 리소스 에셋 설정
	LoadTextures();
	BuildMaterials();
	BuildDescriptorHeaps();

	// 4. 기하 정보 및 정점 버퍼 설정
	BuildShapeGeometry();

	// 5. 프레임 리소스 설정
	BuildFrameResources();
    
    /* 코드 간결성을 위해 생략된 부분이 있습니다. */
    
    return true;
}
```

^f262f3

---

### Stage 1 (Kor)
```cpp
bool D3DApp::Initialize()
{
	// 메인 윈도우 생성 및 초기화
	if(!InitMainWindow())
		return false;

	// Direct3D 디바이스 및 관련 리소스 초기화
	if(!InitDirect3D())
		return false;

    // 윈도우 크기에 맞춰 스왑 체인, 뷰포트, 깊이 버퍼 설정
    OnResize();

	return true;
}
```

```cpp
bool D3DApp::InitDirect3D()
{
	/* 다른 초기화 코드 생략 */

	// GPU 명령을 위한 명령 대기열(commandQueue), 할당자, 명령 목록(commandList) 생성
	CreateCommandObjects();

	// 렌더링된 프레임을 화면에 출력하기 위한 스왑 체인 생성
	CreateSwapChain();

	// 화면 출력용(렌더 대상)과 깊이 정보용 설명자 힙(메모리 영역) 생성
	CreateRtvAndDsvDescriptorHeaps();

	return true;
}
```

^8591c4

---
### Stage 2
```cpp
void Game::BuildRootSignature()
{
    CD3DX12_DESCRIPTOR_RANGE texTable;
    texTable.Init(D3D12_DESCRIPTOR_RANGE_TYPE_SRV, 1, 0);

    CD3DX12_ROOT_PARAMETER slotRootParameter[4];
    slotRootParameter[0].InitAsDescriptorTable(1, &texTable, D3D12_SHADER_VISIBILITY_PIXEL);
    slotRootParameter[1].InitAsConstantBufferView(0);
    slotRootParameter[2].InitAsConstantBufferView(1);
    slotRootParameter[3].InitAsConstantBufferView(2);

    auto staticSamplers = GetStaticSamplers();

    CD3DX12_ROOT_SIGNATURE_DESC rootSigDesc(
        4, slotRootParameter,
        (UINT)staticSamplers.size(), staticSamplers.data(),
        D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT
    );

    ComPtr<ID3DBlob> serializedRootSig = nullptr;
    ComPtr<ID3DBlob> errorBlob = nullptr;
    ThrowIfFailed(D3D12SerializeRootSignature(&rootSigDesc, D3D_ROOT_SIGNATURE_VERSION_1,
        serializedRootSig.GetAddressOf(), errorBlob.GetAddressOf()));

    if (errorBlob)
	    OutputDebugStringA((char*)errorBlob->GetBufferPointer());

    ThrowIfFailed(md3dDevice->CreateRootSignature(
        0,
        serializedRootSig->GetBufferPointer(),
        serializedRootSig->GetBufferSize(),
        IID_PPV_ARGS(mRootSignature.GetAddressOf())));
}
```

^3dc4c3

^fcd7e1
```cpp
void Game::BuildShadersAndInputLayout()
{
    mShaders["standardVS"] = d3dUtil::CompileShader(L"Shaders\\Default.hlsl", nullptr, "VS", "vs_5_1");
    mShaders["opaquePS"]   = d3dUtil::CompileShader(L"Shaders\\Default.hlsl", nullptr, "PS", "ps_5_1");

    mInputLayout =
    {
        { "POSITION",  0, DXGI_FORMAT_R32G32B32_FLOAT, 0,  0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
        { "NORMAL",    0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 12, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
        { "TEXCOORD",  0, DXGI_FORMAT_R32G32_FLOAT,    0, 24, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
    };
}
```

^9e39a9

```cpp
void Game::BuildPSOs()
{
    D3D12_GRAPHICS_PIPELINE_STATE_DESC opaquePsoDesc = {};
    opaquePsoDesc.InputLayout = { mInputLayout.data(), (UINT)mInputLayout.size() };
    opaquePsoDesc.pRootSignature = mRootSignature.Get();
    opaquePsoDesc.VS = 
    { 
        reinterpret_cast<BYTE*>(mShaders["standardVS"]->GetBufferPointer()),
        mShaders["standardVS"]->GetBufferSize() 
    };
    opaquePsoDesc.PS =
    { 
        reinterpret_cast<BYTE*>(mShaders["opaquePS"]->GetBufferPointer()),
        mShaders["opaquePS"]->GetBufferSize() 
    };
    opaquePsoDesc.RasterizerState        = CD3DX12_RASTERIZER_DESC(D3D12_DEFAULT);
    opaquePsoDesc.BlendState             = CD3DX12_BLEND_DESC(D3D12_DEFAULT);
    opaquePsoDesc.DepthStencilState      = CD3DX12_DEPTH_STENCIL_DESC(D3D12_DEFAULT);
    opaquePsoDesc.SampleMask             = UINT_MAX;
    opaquePsoDesc.PrimitiveTopologyType  = D3D12_PRIMITIVE_TOPOLOGY_TYPE_TRIANGLE;
    opaquePsoDesc.NumRenderTargets       = 1;
    opaquePsoDesc.RTVFormats[0]          = mBackBufferFormat;
    opaquePsoDesc.SampleDesc.Count       = m4xMsaaState ? 4 : 1;
    opaquePsoDesc.SampleDesc.Quality     = m4xMsaaState ? (m4xMsaaQuality - 1) : 0;
    opaquePsoDesc.DSVFormat              = mDepthStencilFormat;

    ThrowIfFailed(md3dDevice->CreateGraphicsPipelineState(&opaquePsoDesc, IID_PPV_ARGS(&mOpaquePSO)));
}
```

^95ff5b

### Stage 3
```cpp
void Game::LoadTextures()
{
    std::vector<std::pair<std::string, std::wstring>> textureVec = 
    {
        { "EagleTex",     L"Textures/Eagle.dds" },
        { "RaptorTex",    L"Textures/Raptor.dds" },
        { "DesertTex",    L"Textures/Desert.dds" },
        { "JetColorTex",  L"Textures/Jet.dds" },
        { "Jet2ColorTex", L"Textures/Jet2.dds" },
        { "MissileTex",   L"Textures/Missile.dds" },
        { "TitleTex",     L"Textures/TitleScreen.dds" },
        { "MainMenuTex",  L"Textures/MainMenuScreen.dds" },
        { "PauseTex",     L"Textures/PauseScreen.dds" },
        { "CursorTex",    L"Textures/Cursor.dds" }
    };

    for (size_t i = 0; i < textureVec.size(); ++i)
    {
        const std::string& name = textureVec[i].first;
        const std::wstring& filename = textureVec[i].second;

        auto tex = std::make_unique<Texture>();
        tex->Name = name;
        tex->Filename = filename;

        ThrowIfFailed(DirectX::CreateDDSTextureFromFile12(
            md3dDevice.Get(), mCommandList.Get(),
            tex->Filename.c_str(), tex->Resource, tex->UploadHeap));

        mTextures[name] = std::move(tex);
    }
}

```

^d68072

```cpp
void Game::BuildMaterials()
{
    struct MaterialInfo 
    {
        std::string Name;
        int MatCBIndex;
        int DiffuseSrvHeapIndex;
    };

    std::vector<MaterialInfo> materialVec = 
    { { "Eagle",0,0 }, { "Raptor",1,1 }, { "Desert",2,2 }, { "Missile",3,3 }, { "Title",4,4 }, { "MainMenu",5,5 }, { "Pause",6,6 }, { "Cursor",7,7 } };

    for (auto& m : materialVec)
    {
        auto mat = std::make_unique<Material>();
        mat->Name = m.Name;
        mat->MatCBIndex = m.MatCBIndex;
        mat->DiffuseSrvHeapIndex = m.DiffuseSrvHeapIndex;
        mat->DiffuseAlbedo = XMFLOAT4(1.0f, 1.0f, 1.0f, 1.0f);
        mat->FresnelR0 = XMFLOAT3(0.05f, 0.05f, 0.05f);
        mat->Roughness = 0.2f;
        mMaterials[m.Name] = std::move(mat);
    }
}

```

^1e6763

```cpp
void Game::BuildDescriptorHeaps()
{
    D3D12_DESCRIPTOR_HEAP_DESC heapDesc = {};
    heapDesc.NumDescriptors = 10;
    heapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;
    heapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE;
    ThrowIfFailed(md3dDevice->CreateDescriptorHeap(&heapDesc, IID_PPV_ARGS(&mSrvDescriptorHeap)));

    CD3DX12_CPU_DESCRIPTOR_HANDLE handle(mSrvDescriptorHeap->GetCPUDescriptorHandleForHeapStart());

    D3D12_SHADER_RESOURCE_VIEW_DESC srvDesc = {};
    srvDesc.Shader4ComponentMapping = D3D12_DEFAULT_SHADER_4_COMPONENT_MAPPING;
    srvDesc.ViewDimension = D3D12_SRV_DIMENSION_TEXTURE2D;
    srvDesc.Texture2D.MostDetailedMip = 0;
    srvDesc.Texture2D.ResourceMinLODClamp = 0.0f;

    for (auto key : { "JetColorTex", "Jet2ColorTex", "DesertTex", "MissileTex",
                      "TitleTex", "MainMenuTex", "PauseTex", "CursorTex" })
    {
        auto tex = mTextures[key]->Resource;
        srvDesc.Format = tex->GetDesc().Format;
        srvDesc.Texture2D.MipLevels = tex->GetDesc().MipLevels;
        md3dDevice->CreateShaderResourceView(tex.Get(), &srvDesc, handle);
        handle.Offset(1, mCbvSrvDescriptorSize);
    }
}

```

^a35546

### Stage4
```cpp
void Game::BuildShapeGeometry()
{
	GeometryGenerator geoGen;
	GeometryGenerator::MeshData box = geoGen.CreateBox(1, 0, 1, 1);
	GeometryGenerator::MeshData Eagle = geoGen.CreateMesh("Models/Eagle.txt");
	GeometryGenerator::MeshData Raptor = geoGen.CreateMesh("Models/Raptor.txt");
	GeometryGenerator::MeshData grid = geoGen.CreateGrid(460.0f, 460.0f, 50, 50);

	UINT BoxVertexOffset = 0;
	UINT EagleVertexOffset = (UINT)box.Vertices.size();
	UINT RaptorVertexOffset = EagleVertexOffset + (UINT)Eagle.Vertices.size();
	UINT GridVertexOffset = RaptorVertexOffset + (UINT)Raptor.Vertices.size();

	UINT BoxIndexOffset = 0;
	UINT EagleIndexOffset = (UINT)box.Indices32.size();
	UINT RaptorIndexOffset = EagleIndexOffset + (UINT)Eagle.Indices32.size();
	UINT GridIndexOffset = RaptorIndexOffset + (UINT)Raptor.Indices32.size();

	// Update submeshes
	SubmeshGeometry boxSubmesh;
	boxSubmesh.IndexCount = (UINT)box.Indices32.size();
	boxSubmesh.StartIndexLocation = BoxIndexOffset;
	boxSubmesh.BaseVertexLocation = BoxVertexOffset;

	SubmeshGeometry EagleSubmesh;
	EagleSubmesh.IndexCount = (UINT)Eagle.Indices32.size();
	EagleSubmesh.StartIndexLocation = EagleIndexOffset;
	EagleSubmesh.BaseVertexLocation = EagleVertexOffset;

	SubmeshGeometry RaptorSubmesh;
	RaptorSubmesh.IndexCount = (UINT)Raptor.Indices32.size();
	RaptorSubmesh.StartIndexLocation = RaptorIndexOffset;
	RaptorSubmesh.BaseVertexLocation = RaptorVertexOffset;

	SubmeshGeometry gridSubmesh;
	gridSubmesh.IndexCount = (UINT)grid.Indices32.size();
	gridSubmesh.StartIndexLocation = GridIndexOffset;
	gridSubmesh.BaseVertexLocation = GridVertexOffset;

	// Total vertex count
	UINT TotalVertexCount = (UINT)(box.Vertices.size() + Eagle.Vertices.size() + Raptor.Vertices.size() + grid.Vertices.size());

	// Update vertices
	std::vector<Vertex> vertices(TotalVertexCount);

	size_t BoxVertexCount = box.Vertices.size();
	size_t EagleVertexCount = Eagle.Vertices.size();
	size_t RaptorVertexCount = Raptor.Vertices.size();
	size_t GridVertexCount = grid.Vertices.size();

	// Copy vertices
	for (size_t i = 0; i < BoxVertexCount; ++i)
	{
		vertices[i].Pos = box.Vertices[i].Position;
		vertices[i].Normal = box.Vertices[i].Normal;
		vertices[i].TexC = box.Vertices[i].TexC;
	}

	for (size_t i = 0; i < EagleVertexCount; ++i)
	{
		vertices[BoxVertexCount + i].Pos = Eagle.Vertices[i].Position;
		vertices[BoxVertexCount + i].Normal = Eagle.Vertices[i].Normal;
		vertices[BoxVertexCount + i].TexC = Eagle.Vertices[i].TexC;
	}

	for (size_t i = 0; i < RaptorVertexCount; ++i)
	{
		vertices[BoxVertexCount + EagleVertexCount + i].Pos = Raptor.Vertices[i].Position;
		vertices[BoxVertexCount + EagleVertexCount + i].Normal = Raptor.Vertices[i].Normal;
		vertices[BoxVertexCount + EagleVertexCount + i].TexC = Raptor.Vertices[i].TexC;
	}

	for (size_t i = 0; i < GridVertexCount; ++i)
	{
		auto& p = grid.Vertices[i].Position;
		vertices[BoxVertexCount + EagleVertexCount + RaptorVertexCount + i].Pos = p;
		vertices[BoxVertexCount + EagleVertexCount + RaptorVertexCount + i].Pos.y = GetHillsHeight(p.x, p.z);
		vertices[BoxVertexCount + EagleVertexCount + RaptorVertexCount + i].Normal = GetHillsNormal(p.x, p.z);
		vertices[BoxVertexCount + EagleVertexCount + RaptorVertexCount + i].TexC = grid.Vertices[i].TexC;
	}

	// Update indices
	std::vector<std::uint16_t> indices;

	indices.insert(indices.end(), std::begin(box.GetIndices16()), std::end(box.GetIndices16()));
	indices.insert(indices.end(), std::begin(Eagle.GetIndices16()), std::end(Eagle.GetIndices16()));
	indices.insert(indices.end(), std::begin(Raptor.GetIndices16()), std::end(Raptor.GetIndices16()));
	indices.insert(indices.end(), std::begin(grid.GetIndices16()), std::end(grid.GetIndices16()));

	const UINT vbByteSize = (UINT)vertices.size() * sizeof(Vertex);
	const UINT ibByteSize = (UINT)indices.size() * sizeof(std::uint16_t);

	auto geo = std::make_unique<MeshGeometry>();
	geo->Name = "ShapeGeo";

	// Vertex and index buffers creation
	ThrowIfFailed(D3DCreateBlob(vbByteSize, &geo->VertexBufferCPU));
	CopyMemory(geo->VertexBufferCPU->GetBufferPointer(), vertices.data(), vbByteSize);

	ThrowIfFailed(D3DCreateBlob(ibByteSize, &geo->IndexBufferCPU));
	CopyMemory(geo->IndexBufferCPU->GetBufferPointer(), indices.data(), ibByteSize);

	geo->VertexBufferGPU = d3dUtil::CreateDefaultBuffer(md3dDevice.Get(), mCommandList.Get(), vertices.data(), vbByteSize, geo->VertexBufferUploader);
	geo->IndexBufferGPU  = d3dUtil::CreateDefaultBuffer(md3dDevice.Get(), mCommandList.Get(), indices.data(), ibByteSize, geo->IndexBufferUploader);

	// Geometry properties
	geo->VertexByteStride = sizeof(Vertex);
	geo->VertexBufferByteSize = vbByteSize;
	geo->IndexFormat = DXGI_FORMAT_R16_UINT;
	geo->IndexBufferByteSize = ibByteSize;

	// Draw arguments
	geo->DrawArgs["box"]    = boxSubmesh;
	geo->DrawArgs["Eagle"]  = EagleSubmesh;
	geo->DrawArgs["Raptor"] = RaptorSubmesh;
	geo->DrawArgs["grid"]   = gridSubmesh;

	mGeometries[geo->Name] = std::move(geo);
}
```

^e40791

### Stage 5
```cpp
void Game::BuildFrameResources()
{
	for (int i = 0; i < gNumFrameResources; ++i)
		mFrameResources.push_back(std::make_unique<FrameResource>(md3dDevice.Get(), 1, 200, (UINT)mMaterials.size()));
}
```

^925a50