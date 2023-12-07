---
layout: single
title: "Unreal Engine - Source Control"
categories: Unreal
classes: wide
---

### Unreal Engine - Source Control

**Source Control**    
Unreal에서 제공하는 기능으로 협업에서 파일들을 관리하는 과정을 유용하게 해주는 기능입니다.

Unreal에서 기본적으로 제공하는 기능과, Perforce 또는 SVN을 연동하여 작업하는 방식 두가지가 존재하지만 Perforce 또는 SVN을 사용한 경험이 전무하기 때문에 기본적인 기능에 대해 설명하는 글입니다.

*참조 : Unreal Documentation - Source Control inside Unreal Editor* <https://docs.unrealengine.com/5.0/en-US/using-source-control-in-the-unreal-editor/>

Source Control의 중요 기능은 어느 작업자가 파일을 다루는 동안 다른 작업자가 접근하지 못하게 하거나, 업데이트에 뒤쳐진 파일을 알려주는 등 모든 작업자의 Version이 동일할 수 있도록 관리합니다.

Source Control의 작업 파일 관리 기능은 마치 공유자원 관리를 위한 Mutex처럼 작동합니다.    
Mutex가 파일에 접근 시 Lock을 걸어 다른 Thread가 접근하지 못하게 하는 것 처럼, Source Control은 한 작업자가 Check Out을 통해 파일에 접근하면    
작업자 이외의 사람들에겐 해당 파일이 잠금 상태가 되어 접근할 수 없습니다.

Asset의 Source Control에 접근하기 위해선 Content Brower에서 우클릭->Source Control을 통해 접근할 수 있으며, 이 때 다음 중 하나의 심볼을 통해 현재 Asset의 상태를 나타냅니다.

![](/assets/images/Unreal/SourceControlState.PNG)
*Asset State 표 / 출처 - Unreal Documentation*

표에 나와있는 심볼을 통해 Asset이 Check Out상태인지, 새로운 버전의 Asset이 Source Control 내부에 존재하는지, Asset 자체가 Source Control에 존재하지 않는지 등을 알 수 있습니다.

만약 작업자가 작업을 완료하였다면, 작업이 완료된 Asset들을 다시 Source Control에 업로드하여 마치 Git에서 Submit과 같이 다른 작업자에게 작업한 파일에 대한 Comment와 함께 업로드할 수 있으며,  다른 작업자들은 업로드가 완료된 파일들에 대해 ! 심볼과 함께 새로운 버전의 Asset을 인지하고 Sync라는 기능을 통해 모든 작업자의 Asset에 대한 통일성을 가질 수 있습니다.

파일의 삭제 과정은 기존에 Asset을 삭제하는 과정과 동일하지만, 삭제 후 해당 Asset의 자리는 Reference 등을 관리하기 위해 숨겨진 Redirector가 생성되게 되며 이는 Content Browser의 Fix Up Redirectors in Folder 기능을 통해 주기적으로 정리해 주어야 합니다.

실제 현업에서는 강력한 기능을 자랑하는 Perforce등을 연동하여 사용하겠지만, 포트폴리오 제작등을 위한 소규모 팀 프로젝트 등에서는 Source Control을 통해 협업 파일 관리에 대한 용이성을 확보할 수 있을 것입니다.
