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
* GDI +
GDI+란 GDI의 업그레이드 버전으로 복잡하고 섬세한 그래픽을 출력할 수 있는 기존의 기능을 최적화한 새로운 출력 모듈입니다.
* GDI+를 이용한 펜과 지우개를 이용한 그림판 만들기
이 프로젝트에선 왼쪽 마우스는 펜의 기능을 가지고, 오른쪽 마우스는 지우개 기능을 가지는 프로젝트를 만들었습니다.
먼저 툴바를 만들고, 펜과 지우개의 커서 리소스를 만든 뒤에 펜과 지우개의 크기를 정하는 대화상자를 만들었습니다.
![image](https://github.com/user-attachments/assets/6e6701a2-bb1f-4502-bf14-2e9c39a9bf30)
그 후에 메뉴들에 대한 함수들을 만들고, 마우스가 눌렸을 때와 움직였을 때의 기능을 구현했습니다.
``` ruby
void CPractice7bView::OnLButtonDown(UINT nFlags, CPoint point)
{
	// TODO: 여기에 메시지 처리기 코드를 추가 및/또는 기본값을 호출합니다.
	m_ptPrev = point;
	HCURSOR hCursor = AfxGetApp()->LoadCursor(IDC_CURSOR_PEN);
	SetCursor(hCursor);
	CView::OnLButtonDown(nFlags, point);
}


void CPractice7bView::OnRButtonDown(UINT nFlags, CPoint point)
{
	// TODO: 여기에 메시지 처리기 코드를 추가 및/또는 기본값을 호출합니다.
	m_ptPrev = point;
	HCURSOR hCursor = AfxGetApp()->LoadCursor(IDC_CURSOR_ERASER);
	SetCursor(hCursor);
	CView::OnRButtonDown(nFlags, point);
}


void CPractice7bView::OnMouseMove(UINT nFlags, CPoint point)
{
	// TODO: 여기에 메시지 처리기 코드를 추가 및/또는 기본값을 호출합니다.
	CClientDC dc(this);
	Graphics graphics(dc);
	Gdiplus::Color clr; //GDI+로의 색상변경
	clr.SetFromCOLORREF(m_colorPen);

	if (nFlags == MK_LBUTTON)
	{
		HCURSOR hCursor = AfxGetApp()->LoadCursorW(IDC_CURSOR_PEN);
		SetCursor(hCursor);
		Pen pen(Color(clr), m_nPenSize);
		graphics.DrawLine(&pen, m_ptPrev.x, m_ptPrev.y, point.x, point.y);
		m_ptPrev = point;
	}

	if (nFlags == MK_RBUTTON)
	{
		HCURSOR hCursor = AfxGetApp()->LoadCursorW(IDC_CURSOR_ERASER);
		SetCursor(hCursor);
		Pen pen(Color(255, 255, 255), m_nEraserSize);
		graphics.DrawLine(&pen, m_ptPrev.x, m_ptPrev.y, point.x, point.y);
		m_ptPrev = point;
	}

	CView::OnMouseMove(nFlags, point);
}
```
왼쪽 마우스를 누르면 커서가 바뀌고, 이동시키면 펜의 색상이 GDI+ 형식으로 바뀌며 오른쪽 마우스를 누르면 커서가 바뀌고, 이동시키면 지우개의 색상이 흰색이 됩니다.
출력화면
![image](https://github.com/user-attachments/assets/94852b51-c1bd-4a39-a4a8-23c21de78ca5)
실행하면 예시로 좌클릭으로 그림을 그리고, 우클릭으로 일부를 지운 화면입니다.

--------------------------
* 컨트롤 및 리소스
리스트 컨트롤은 주로 GUI 애플리케이션에서 데이터를 보기 좋은 방식으로 정리하여 표시하기 위해 사용되는 컨트롤로, 데이터를 표 형식으로 나열합니다.
* 대화상자에 리스트 컨트롤을 사용한 프로젝트
이 프로젝트에서는 대화상자에 리스트 컨트롤을 만들고, 데이터의 추가, 수정, 삭제를 할 수 있습니다.
먼저 대화상자를 만들어준뒤 속성을 변경해줍니다.
![image](https://github.com/user-attachments/assets/ce2dbd49-0b79-4bdd-8ce2-59a8652713c0)
그 뒤 데이터를 추가하는 버튼을 구현해주었습니다.
``` ruby
void CPractice8aDlg::OnClickedButtonInsert()
{
	// TODO: 여기에 컨트롤 알림 처리기 코드를 추가합니다.
	int nCount = m_listStudent.GetItemCount();
	CString strCount;
	UpdateData(TRUE);

	if (!m_strDept.IsEmpty() && !m_strID.IsEmpty() && !m_strName.IsEmpty())
	{
		strCount.Format(_T("%d"), nCount + 1);
		m_listStudent.InsertItem(nCount, strCount);
		m_listStudent.SetItem(nCount, 1, LVIF_TEXT, m_strDept, 0, 0, 0, 0);
		m_listStudent.SetItem(nCount, 2, LVIF_TEXT, m_strID, 0, 0, 0, 0);
		m_listStudent.SetItem(nCount, 3, LVIF_TEXT, m_strName, 0, 0, 0, 0);

		m_strDept.Empty();
		m_strID.Empty();
		m_strName.Empty();

		//수정/삭제 버튼을 비활성화 시킨다.
		((CButton*)GetDlgItem(IDC_BUTTON__MODIFY))->EnableWindow(FALSE);
		((CButton*)GetDlgItem(IDC_BUTTON_DELETE))->EnableWindow(FALSE);
		UpdateData(FALSE);
	}

	else
	{
		MessageBox(_T("모든 항목을 입력해 주세요."), _T("잠깐"), MB_OK);
	}
}
```
데이터가 모두 입력이 되어있다면 리스트 컨트롤에 새로운 항목을 추가한 뒤 입력 필드를 초기화합니다.
그 뒤 데이터를 수정하는 버튼을 구현하였습니다.
``` ruby
void CPractice8aDlg::OnClickedButtonModify()
{
	// TODO: 여기에 컨트롤 알림 처리기 코드를 추가합니다.
	UpdateData(TRUE);
	CString strDept, strID, strName, strIndex;
	strDept = m_strDept;
	strID = m_strID;
	strName = m_strName;

	if (m_nSelectedItem >= 0)
	{

		if (!m_strDept.IsEmpty() && !m_strID.IsEmpty() && !m_strName.IsEmpty())
		{
			strIndex.Format(_T("%d"), m_nSelectedItem + 1);
			m_listStudent.SetItem(m_nSelectedItem, 0, LVIF_TEXT, strIndex, 0, 0, 0, 0);
			m_listStudent.SetItem(m_nSelectedItem, 1, LVIF_TEXT, strDept, 0, 0, 0, 0);
			m_listStudent.SetItem(m_nSelectedItem, 2, LVIF_TEXT, strID, 0, 0, 0, 0);
			m_listStudent.SetItem(m_nSelectedItem, 3, LVIF_TEXT, strName, 0, 0, 0, 0);

			m_strDept.Empty();
			m_strID.Empty();
			m_strName.Empty();

			((CButton*)GetDlgItem(IDC_BUTTON__MODIFY))->EnableWindow(FALSE);
			((CButton*)GetDlgItem(IDC_BUTTON_DELETE))->EnableWindow(FALSE);
			UpdateData(FALSE);
		}

		else
		{
			MessageBox(_T("모든 항목을 입력해 주세요."), _T("잠깐"), MB_OK);
		}
	}

	else
	{
		MessageBox(_T("아이템을 선택하지 않았습니다."), _T("잠깐"), MB_OK);
	}
}
```
리스트 컨트롤에서 데이터를 클릭한 뒤 에딧 컨트롤에서 데이터를 수정 한 후 수정 버튼을 누르면 에딧 컨트롤에서 수정된 데이터로 리스트 컨트롤의 해당 아이템을 업데이트합니다.
그 후 데이터를 삭제하는 코드를 구현했습니다.
``` ruby
void CPractice8aDlg::OnClickedButtonDelete()
{
	// TODO: 여기에 컨트롤 알림 처리기 코드를 추가합니다.
	if (m_nSelectedItem >= 0)
	{
		m_listStudent.DeleteItem(m_nSelectedItem);

		for (int i = m_nSelectedItem - 1; i < m_listStudent.GetItemCount(); i++)
		{
			CString strIndex;
			strIndex.Format(_T("%d"), i + 1);
			m_listStudent.SetItemText(i, 0, strIndex);
		}

		m_strDept.Empty();
		m_strID.Empty();
		m_strName.Empty();
		m_strSelectedItem.Empty();

		((CButton*)GetDlgItem(IDC_BUTTON__MODIFY))->EnableWindow(FALSE);
		((CButton*)GetDlgItem(IDC_BUTTON_DELETE))->EnableWindow(FALSE);
		UpdateData(FALSE);
	}

	else
	{
		MessageBox(_T("아이템을 선택하지 않았습니다."), _T("잠깐"), MB_OK);
	}
}
```
리스트 컨트롤에서 데이터를 클릭한 뒤 삭제 버튼을 누르면 리스트 컨트롤에서 삭제됩니다.
그 후 다시 쓰기 버튼을 구현했습니다.
``` ruby
void CPractice8aDlg::OnClickedButtonReset()
{
	// TODO: 여기에 컨트롤 알림 처리기 코드를 추가합니다.
	m_strDept.Empty();
	m_strID.Empty();
	m_strName.Empty();
	UpdateData(FALSE);
}
```
다시 쓰기 버튼을 누르면 현재 입력한 데이터를 모두 초기화 합니다.
* 트리 컨트롤
트리 컨트롤을 만들고 수정하고 삭제합니다
![image](https://github.com/user-attachments/assets/056cd1f4-36d0-4fb3-b3b4-abcda8f59530)
대화상자를 만든 뒤에
``` ruby
void CPractice8bDlg::OnClickedButtonInsert()
{
	// TODO: 여기에 컨트롤 알림 처리기 코드를 추가합니다.
	UpdateData(TRUE);
	//에러 처리 - 입력할 텍스트가 비어 있나 검사한다.
	if (!m_strNodeText.IsEmpty())
	{
		m_treeControl.InsertItem(m_strNodeText, m_hSelectedNode, TVI_LAST);
		m_treeControl.Expand(m_hSelectedNode, TVE_EXPAND);
	}
	else
	{
		AfxMessageBox(_T("입력 노드의 텍스트를 입력하세요."));
	}
	//Edit Box의 텍스트를 비운다
	m_strNodeText.Empty();
	UpdateData(FALSE);
}


void CPractice8bDlg::OnClickedButtonModify()
{
	// TODO: 여기에 컨트롤 알림 처리기 코드를 추가합니다.
	UpdateData(TRUE);
	//입력할 텍스트가 비어 있는지 검사
	if (!m_strNodeText.IsEmpty())
	{
		if (m_hSelectedNode != m_hRoot)
		{
			//선택된 아이템의 텍스트를 수정한다.
			m_treeControl.SetItemText(m_hSelectedNode, m_strNodeText);
			//현재 선택된 아이템의 이름을 표현하는 Edit Control의 내용도 수정해 준다.
			m_strSelectedNode = m_strNodeText;
		}
		else
		{
			AfxMessageBox(_T("루트 노드는 수정해서는 안 됩니다."));
		}
	}
	else
	{
		AfxMessageBox(_T("수정 노드의 텍스트를 입력하세요."));
	}
	m_strNodeText.Empty();
	UpdateData(FALSE);
}


void CPractice8bDlg::OnClickedButtonDelete()
{
	// TODO: 여기에 컨트롤 알림 처리기 코드를 추가합니다.
	if (m_hSelectedNode != m_hRoot)
	{
		if (MessageBox(_T("정말 삭제하시겠습니까?"), _T("삭제 경고"), MB_YESNO) == IDYES)
		{
			m_treeControl.DeleteItem(m_hSelectedNode);
		}
	}
	else
	{
		AfxMessageBox(_T("루트 노드는 삭제해서는 안 됩니다."));
	}
}


void CPractice8bDlg::OnClickedCheckExpand()
{
	// TODO: 여기에 컨트롤 알림 처리기 코드를 추가합니다.
	m_bChecked = !m_bChecked;
	m_treeControl.Expand(m_hRoot, TVE_TOGGLE);
	m_hSelectedNode = m_hRoot;
	m_strSelectedNode = _T("루트 노드");
	UpdateData(FALSE);
}
```
트리를 구현합니다
![image](https://github.com/user-attachments/assets/bee62579-f9de-4161-8a0e-191fe62e927c)
실행화면입니다.

-----------
* 탭 컨트롤 활용한 도형 출력 프로젝트
먼저 대화상자를 구성합니다.
![image](https://github.com/user-attachments/assets/30a92165-38a3-4216-bfdd-cb2a8dfa8f84)
그 뒤 도형을 출력하고 설정하는 것을 구현합니다
``` ruby
void CPractiec9aDlg::OnSelchangeTabSelection(NMHDR *pNMHDR, LRESULT *pResult)
{
	// TODO: 여기에 컨트롤 알림 처리기 코드를 추가합니다.
	int nSelection = m_tabSelection.GetCurSel();
	switch (nSelection)
	{
	case 0 :
		m_dlgObject.ShowWindow(SW_SHOW);
		m_dlgColor.ShowWindow(SW_HIDE);
		m_dlgRatio.ShowWindow(SW_HIDE);
		break;
	case 1 :
		m_dlgObject.ShowWindow(SW_HIDE);
		m_dlgColor.ShowWindow(SW_SHOW);
		m_dlgRatio.ShowWindow(SW_HIDE);
		break;
	case 2 :
		m_dlgObject.ShowWindow(SW_HIDE);
		m_dlgColor.ShowWindow(SW_HIDE);
		m_dlgRatio.ShowWindow(SW_SHOW);
		break;
	}
	*pResult = 0;
}

BOOL CColorDlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// TODO:  여기에 추가 초기화 작업을 추가합니다.
	m_colorObject = RGB(255, 0, 0);
	m_sliderRed.SetRange(0, 255);
	m_sliderGreen.SetRange(0, 255);
	m_sliderBlue.SetRange(0, 255);
	m_sliderRed.SetPos(255);
	m_sliderGreen.SetPos(0);
	m_sliderBlue.SetPos(0);
	m_nRed = 255;
	m_nGreen = 0;
	m_nBlue = 0;
	UpdateData(FALSE);
	return TRUE;  // return TRUE unless you set the focus to a control
				  // 예외: OCX 속성 페이지는 FALSE를 반환해야 합니다.
}


void CColorDlg::OnHScroll(UINT nSBCode, UINT nPos, CScrollBar* pScrollBar)
{
	// TODO: 여기에 메시지 처리기 코드를 추가 및/또는 기본값을 호출합니다.
	//탭 컨트롤을 포함하고 있는 메인 대화상자의 인스턴스를 얻는다.
	CPractiec9aDlg* pMainDlg = (CPractiec9aDlg*)AfxGetMainWnd();
	if (pScrollBar->GetSafeHwnd() == m_sliderRed.m_hWnd)
	{
		m_nRed = m_sliderRed.GetPos();
	}
	else if (pScrollBar->GetSafeHwnd() == m_sliderGreen.m_hWnd)
	{
		m_nGreen = m_sliderGreen.GetPos();
	}
	else if (pScrollBar->GetSafeHwnd() == m_sliderBlue.m_hWnd)
	{
		m_nBlue = m_sliderBlue.GetPos();
	}
	else
	{
		return;
	}
	m_colorObject = RGB(m_nRed, m_nGreen, m_nBlue);
	UpdateData(FALSE);
	pMainDlg->UpdateDrawing();;
	CDialogEx::OnHScroll(nSBCode, nPos, pScrollBar);
}

BOOL CRatioDlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// TODO:  여기에 추가 초기화 작업을 추가합니다.
	m_bSameRatio = TRUE;
	((CButton*)GetDlgItem(IDC_CHECK_SAME_RATIO))->SetCheck(TRUE);
	m_sliderHorizontal.SetRange(0, 100);
	m_sliderVertical.SetRange(0, 100);
	m_sliderHorizontal.SetPos(50);
	m_sliderVertical.SetPos(50);
	m_nHorizontal = 50;
	m_nVertical = 50;
	m_nCurHScale = 50;
	m_nCurVScale = 50;
	UpdateData(FALSE);
	return TRUE;  // return TRUE unless you set the focus to a control
				  // 예외: OCX 속성 페이지는 FALSE를 반환해야 합니다.
}


void CRatioDlg::OnHScroll(UINT nSBCode, UINT nPos, CScrollBar* pScrollBar)
{
	// TODO: 여기에 메시지 처리기 코드를 추가 및/또는 기본값을 호출합니다.
	CPractiec9aDlg* pMainDlg = (CPractiec9aDlg*)AfxGetMainWnd();
	m_nCurHScale = m_sliderHorizontal.GetPos();
	m_nCurVScale = m_sliderVertical.GetPos();
	if (pScrollBar->GetSafeHwnd() == m_sliderHorizontal.m_hWnd)
	{
		if (m_bSameRatio == TRUE)
		{
			m_sliderVertical.SetPos(m_nCurHScale);
		}
	}
	else if (pScrollBar->GetSafeHwnd() == m_sliderVertical.m_hWnd)
	{
		if (m_bSameRatio == TRUE)
		{
			m_sliderHorizontal.SetPos(m_nCurVScale);
		}
	}
	else
	{
		return;
	}
	m_nHorizontal = m_nCurHScale;
	m_nVertical = m_nCurVScale;
	UpdateData(FALSE);
	pMainDlg->UpdateDrawing();
	CDialogEx::OnHScroll(nSBCode, nPos, pScrollBar);
}


void CRatioDlg::OnClickedCheckSameRatio()
{
	// TODO: 여기에 컨트롤 알림 처리기 코드를 추가합니다.
	UpdateData(TRUE);
	CPractiec9aDlg* pMainDlg = (CPractiec9aDlg*)AfxGetMainWnd();
	m_bSameRatio = !m_bSameRatio;
	if (m_bSameRatio == TRUE)
	{
		if (m_nCurHScale > m_nCurVScale)
		{
			m_nHorizontal = m_sliderHorizontal.GetPos();
			m_nVertical = m_nHorizontal;
		}
		else
		{
			m_nVertical = m_sliderVertical.GetPos();
			m_nHorizontal = m_nVertical;
		}
		m_nCurHScale = m_nHorizontal;
		m_nCurVScale = m_nVertical;
		m_sliderHorizontal.SetPos(m_nCurVScale);
		m_sliderVertical.SetPos(m_nCurVScale);
		UpdateData(FALSE);
	}
	pMainDlg->UpdateDrawing();
}

BOOL CObjectDlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// TODO:  여기에 추가 초기화 작업을 추가합니다.
	m_nSelObject = 1;
	((CButton*)GetDlgItem(IDC_RADIO_RECT))->SetCheck(TRUE);
	return TRUE;  // return TRUE unless you set the focus to a control
				  // 예외: OCX 속성 페이지는 FALSE를 반환해야 합니다.
}


void CObjectDlg::OnRadioRect()
{
	// TODO: 여기에 명령 처리기 코드를 추가합니다.
	//탭 컨트롤을 포함하고 있는 메인 대화상자의 인스턴스를 얻는다.
	CPractiec9aDlg* pMainDlg = (CPractiec9aDlg*)AfxGetMainWnd();
	m_nSelObject = 1;
	pMainDlg->UpdateDrawing();
}


void CObjectDlg::OnRadioCircle()
{
	// TODO: 여기에 명령 처리기 코드를 추가합니다.
	//탭 컨트롤을 포함하고 있는 메인 대화상자의 인스턴스를 얻는다.
	CPractiec9aDlg* pMainDlg = (CPractiec9aDlg*)AfxGetMainWnd();
	m_nSelObject = 2;
	pMainDlg->UpdateDrawing();
}
```
색상,도형,비율 선택하는 대화상자를 출력하는 코드와 각각의 기능을 구현하는 코드입니다.
출력화면
![image](https://github.com/user-attachments/assets/bec88d1c-1397-4c35-8351-029ef1269a45)
색을 바꾸고 도형을 바꾸고 크기를 바꾼 모습입니다

-------------
