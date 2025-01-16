# mfcstudy
* 사용자 인터페이스
메뉴는 윈도우에서 가장 보편적으로 사용되는 사용자 인터페이스로, 사용자가 프로그램에 기능을 수행하도록 명령하는 기능이 있다.
메뉴의 형태는 크게 3가지로, 애플리케이션 상단에 여래 개의 카테고리가 일렬로 늘어서 있는 형태인 풀다운 메뉴, 메뉴 항목의 오른쪽에 또 하나의 서브 메뉴가 나타나는 캐스케이딩 메뉴, 애플리케이션 영역의 중간에서 자유롭게 튀어나오는 팝업 메뉴가 있습니다.
* GDI와 DC
윈도우에서의 그래픽 출력은 크게 GDI와 DC가 있는데 이때 GDI란 그래픽 디바이스 인터페이스의 약자로 윈도우가 하드웨어를 제어할 수 있도록 애플리케이션에 제공하는 모든 기능을 일컷는 말입니다.
DC란 디바이스 컨텍스트의 약자로 그래픽 객체들의 속성과 그래픽 모드를 정의하는 자료의 집합체입니다.
* 직선, 곡선, 도형을 그리는 프로젝트
이 프로젝트는 마우스로 직선, 원, 베지어 곡선을 그리는 프로젝트입니다.
먼저, 메뉴화면을 만들어줍니다.
![image](https://github.com/user-attachments/assets/63893b41-d4c7-4915-b265-0fba302dcc39)
그리기, 색상, 무늬 메뉴를 만들고 그안에 부메뉴들을 만들었습니다.
그리고 단축기를 설정해줍니다.
![image](https://github.com/user-attachments/assets/19421ad9-793f-4079-9c33-d04435359c4c)
ID항목에서 단축기를 설정하고자 하는 메뉴 항목에 오른쪽에 있는 화살표로 ID를 선택하고, 속성에서 단축키를 사용할 키와 보조키를 설정했습니다.
선택된 메뉴에 체크표시를 하게 했습니다.
``` ruby
void CPractice6aView::OnUpdateLine(CCmdUI *pCmdUI)
{
	// TODO: 여기에 명령 업데이트 UI 처리기 코드를 추가합니다.
	//직선 그리기 모드이면 메뉴에 체크 표시
	pCmdUI->SetCheck(m_nDrawMode == LINE_MODE ? 1 : 0);
}


void CPractice6aView::OnUpdateBezier(CCmdUI *pCmdUI)
{
	// TODO: 여기에 명령 업데이트 UI 처리기 코드를 추가합니다.
	//다각형 그리기 모드이면 메뉴에 체크 표시
	pCmdUI->SetCheck(m_nDrawMode == BEZIER_MODE ? 1 : 0);
}


void CPractice6aView::OnUpdateEllipse(CCmdUI *pCmdUI)
{
	// TODO: 여기에 명령 업데이트 UI 처리기 코드를 추가합니다.
	//원 그리기 모드이면 메뉴에 체크 표시
	pCmdUI->SetCheck(m_nDrawMode == ELLIPSE_MODE ? 1 : 0);
}
```
만약 직선 그리기 모드라면 true를 반환해 체크 표시를 합니다.
그 뒤 툴바를 설정해준 뒤 직선,곡선을 그리게 하였습니다.
``` ruby
void CPractice6aView::OnLButtonDown(UINT nFlags, CPoint point)
{
	// TODO: 여기에 메시지 처리기 코드를 추가 및/또는 기본값을 호출합니다.
	//각 변수들 초기화
	if (m_bFirst)
	{
		m_nCount = 0;
		for (int i = 0; i < 50; i++)
		{
			m_ptData[i] = 0;
		}
		Invalidate(true); //화면 갱신
	}

	switch (m_nDrawMode)
	{
	case LINE_MODE : //직선 그리기
	case ELLIPSE_MODE : //원 그리기
		m_bLButtonDown = true; //왼쪽 버튼 눌림
		m_ptStart = m_ptPrev = point; //시작점과 이전 점에 현재 점을 저장
		m_bFirst = false; //처은 그리는 것 -> false
		break;

	case BEZIER_MODE : //컨트롤 폴리곤 그리기
		CClientDC dc(this); //클라이언트 영역 DC 얻음
		dc.SelectObject(GetStockObject(GRAY_BRUSH)); 
		m_ptStart = m_ptPrev = point; //시작점과 이전 점에 현재 점을 저장
		dc.Ellipse(point.x - 4, point.y - 4, point.x + 4, point.y + 4);
		if (m_bFirst == true)
		{
			m_bFirst = false; //처음 그리는 것 -> false
		}
		m_ptData[m_nCount] = point; //현재 점을 저장
		m_nCount++; //카운트 증가
		break;
	}

	RECT rectClient; //구조체 변수 선언
	SetCapture(); //마우스 캡쳐
	GetClientRect(&rectClient); //클라이언트 영역 받음
	ClientToScreen(&rectClient); //스크린 좌표계 변환
	::ClipCursor(&rectClient); //마우스 이동범위를 클라이언트 영역으로 제한
	CView::OnLButtonDown(nFlags, point);
}
```
좌클릭을 했을 때의 코드로 먼저 각 변수들을 초기화하며 화면을 초기화 하고 펜을 생성합니다.
그 후 마우스를 움직일 떄의 코드를 구현했습니다.
``` ruby
void CPractice6aView::OnMouseMove(UINT nFlags, CPoint point)
{
	// TODO: 여기에 메시지 처리기 코드를 추가 및/또는 기본값을 호출합니다.
	CClientDC dc(this); //클라이언트 객체 얻음
	CPen pen, *oldpen;
	pen.CreatePen(PS_SOLID, 1, m_colorPen); //pen 객체 생성
	oldpen = dc.SelectObject(&pen); //pen 객체 등록
	dc.SetROP2(R2_NOTXORPEN); //R2_NOTXORPEN으로 설정
	CBrush brush, *oldbrush;

	if (m_bHatch)
	{
		brush.CreateHatchBrush(m_nHatchStyle, m_colorBrush); //Hatch Brush 객체 생성
	}

	else
	{
		brush.CreateSolidBrush(m_colorBrush); //Solod Brush 객체 생성
	}
	oldbrush = dc.SelectObject(&brush); //brush 객체 등록

	switch (m_nDrawMode)
	{
	case LINE_MODE :

		if (m_bLButtonDown)
		{
			dc.MoveTo(m_ptStart);
			dc.LineTo(m_ptPrev); //이전 직선 지움
			dc.MoveTo(m_ptStart);
			dc.LineTo(point); //현재 직선 그림
			m_ptPrev = point; //이전 점에 현재 점을 저장
		}

		break;

	case ELLIPSE_MODE : //원 그리기

		if (m_bLButtonDown)
		{
			dc.Ellipse(m_ptStart.x, m_ptStart.y, m_ptPrev.x, m_ptPrev.y);
			dc.Ellipse(m_ptStart.x, m_ptStart.y, point.x, point.y);
			m_ptPrev = point; //이전 점에 현재 점을 저장
		}

		break;

	case BEZIER_MODE : //컨트롤 폴리곤 그리기

		if (!m_bFirst)
		{
			dc.MoveTo(m_ptStart); //시작점에서 이전 점까지 선을 지움
			dc.LineTo(m_ptPrev);
			dc.MoveTo(m_ptStart); //시작점에서 현재 점까지 선을 그림
			dc.LineTo(point);
			m_ptPrev = point; //이전 점에 현재 점을 저장
		}

		break;
	}

	dc.SelectObject(oldpen); //이전 pen으로 설정
	dc.SelectObject(oldbrush); //이전 brush로 설정
	pen.DeleteObject(); //pen 객체 삭제
	brush.DeleteObject(); //brush 객체 삭제

	//메인프레임의 포인터 얻음
	CMainFrame *pFrame = (CMainFrame *)AfxGetMainWnd();
	CString strPoint;
	strPoint.Format(_T("마우스 위치 x : %d, y : %d"), point.x, point.y);

	//새로 추가한 팬에 마우스 위치 출력
	pFrame->m_wndStatusBar.SetPaneText(1, strPoint);
	CView::OnMouseMove(nFlags, point);
}
```
먼저 펜과 브러시를 생성하고 그리기 모드에 따라 그리게 하는 코드입니다.
이 외에도 색을 변경하는 등의 코드를 구현하였습니다.
출력화면
![image](https://github.com/user-attachments/assets/5d21bcff-7ad4-4fd2-89d4-ad597358e247)
출력하면 예시로 색상 메뉴에서 면 색상을 변경하고 원 툴바를 누른디 마우스로 원을 그렸습니다.

---------------------------------------
* 
