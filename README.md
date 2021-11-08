<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>CO-VADE 시스템</title>
</head>
<body>
	
	<div id="map" style="width:100%;height:100vh;"></div>

	<script src="https://dapi.kakao.com/v2/maps/sdk.js?appkey=b77f7b6983dfef28e72cdef742180c0f&libraries=services,clusterer,drawing"></script>
	<script>



		// 지도 설정 값
		var mapContainer = document.getElementById('map'), // 지도를 표시할 div 
		    mapOption = {
		        center: new kakao.maps.LatLng(37.262136808290435, 127.0412112365976), // 지도의 중심좌표
		        level: 3, // 지도의 확대 레벨
		        mapTypeId : kakao.maps.MapTypeId.ROADMAP // 지도종류
		    }; 

		// 지도를 생성한다 
		var map = new kakao.maps.Map(mapContainer, mapOption); 

		// 지도에 확대 축소 컨트롤을 생성한다
		var zoomControl = new kakao.maps.ZoomControl();

		// 지도의 우측에 확대 축소 컨트롤을 추가한다
		map.addControl(zoomControl, kakao.maps.ControlPosition.RIGHT);
		



		//코로나19 확진자 방문위치 마커(데이터)
		// 마커 클러스터러를 생성합니다 
		var clusterer = new kakao.maps.MarkerClusterer({
			map: map, // 마커들을 클러스터로 관리하고 표시할 지도 객체 
			averageCenter: true, // 클러스터에 포함된 마커들의 평균 위치를 클러스터 마커 위치로 설정 
			minLevel: 10 // 클러스터 할 최소 지도 레벨 
		});

		var mapdata = [
			[37.26237589427931, 127.04137216928672,'<div style="padding: 5px">매탄위브하늘채</div>'],
			[37.26641044898692, 127.03445646623697,'<div style="padding: 5px">뉴코아아울렛</div>'],
			[37.263473211476445, 127.03205320687009,'<div style="padding: 5px">동수원CGV</div>'],
			[37.26207286026976, 127.02977869376929,'<div style="padding: 5px">수원시청역</div>'],
		];

		var markers = [];

		for(var i = 0;i<mapdata.length;i++){
			// 지도에 마커를 생성하고 표시한다
			var imageSrc = "patient.png", // 마커이미지의 주소입니다    
				imageSize = new kakao.maps.Size(25, 35), // 마커이미지의 크기입니다
				imageOption = {offset: new kakao.maps.Point(27, 69)}; // 마커이미지의 옵션입니다. 마커의 좌표와 일치시킬 이미지 안에서의 좌표를 설정합니다.
				
			// 마커의 이미지정보를 가지고 있는 마커이미지를 생성합니다
			var markerImage = new kakao.maps.MarkerImage(imageSrc, imageSize, imageOption),
				markerPosition = new kakao.maps.LatLng(37.54699, 127.09598); // 마커가 표시될 위치입니다

			// 마커를 생성하고 지도에 표시합니다
			var marker = new kakao.maps.Marker({
				map: map,
				image: markerImage, // 마커이미지 설정 
				position: new kakao.maps.LatLng(mapdata[i][0], mapdata[i][1])
			});
			
			// 인포윈도우를 생성합니다
			var infowindow = new kakao.maps.InfoWindow({ 
				content : mapdata[i][2] 
			});
			
			// 마커 위에 인포윈도우를 표시합니다. 두번째 파라미터인 marker를 넣어주지 않으면 지도 위에 표시됩니다
			markers.push(marker);

			// 마커에 mouseover 이벤트와 mouseout 이벤트를 등록합니다
			// 이벤트 리스너로는 클로저를 만들어 등록합니다 
			// for문에서 클로저를 만들어 주지 않으면 마지막 마커에만 이벤트가 등록됩니다
			kakao.maps.event.addListener(marker, 'mouseover', makeOverListener(map, marker, infowindow));
			kakao.maps.event.addListener(marker, 'mouseout', makeOutListener(infowindow));
		}

		// 클러스터러에 마커들을 추가합니다
        clusterer.addMarkers(markers);

		// 인포윈도우를 표시하는 클로저를 만드는 함수입니다 
		function makeOverListener(map, marker, infowindow) {
			return function() {
				infowindow.open(map, marker);
			};
		}

		// 인포윈도우를 닫는 클로저를 만드는 함수입니다 
		function makeOutListener(infowindow) {
			return function() {
				infowindow.close();
			};
		}



		//코로나 선별 진료소 마커(검색)
		// 마커를 클릭하면 장소명을 표출할 인포윈도우 입니다
		var infowindow = new kakao.maps.InfoWindow({zIndex:1});

		// 장소 검색 객체를 생성합니다
		var ps = new kakao.maps.services.Places(); 

		// 키워드로 장소를 검색합니다
		ps.keywordSearch('수원코로나검사소' , placesSearchCB); 

		// 키워드 검색 완료 시 호출되는 콜백함수 입니다
		function placesSearchCB (data, status, pagination) {
			if (status === kakao.maps.services.Status.OK) {

				// 검색된 장소 위치를 기준으로 지도 범위를 재설정하기위해
				// LatLngBounds 객체에 좌표를 추가합니다
				var bounds = new kakao.maps.LatLngBounds();

				for (var i=0; i<data.length; i++) {
					displayMarker(data[i]);    
					bounds.extend(new kakao.maps.LatLng(data[i].y, data[i].x));
				}       

				// 검색된 장소 위치를 기준으로 지도 범위를 재설정합니다
				map.setBounds(bounds);
			} 
		}

		// 지도에 마커를 표시하는 함수입니다
		function displayMarker(place) {
			var imageSrc = "hospital.png", // 마커이미지의 주소입니다    
				imageSize = new kakao.maps.Size(25, 35), // 마커이미지의 크기입니다
				imageOption = {offset: new kakao.maps.Point(27, 69)}; // 마커이미지의 옵션입니다. 마커의 좌표와 일치시킬 이미지 안에서의 좌표를 설정합니다.
				
			// 마커의 이미지정보를 가지고 있는 마커이미지를 생성합니다
			var markerImage = new kakao.maps.MarkerImage(imageSrc, imageSize, imageOption),
				markerPosition = new kakao.maps.LatLng(37.54699, 127.09598); // 마커가 표시될 위치입니다

			// 마커를 생성하고 지도에 표시합니다
			var marker = new kakao.maps.Marker({
				map: map,
				image: markerImage, // 마커이미지 설정 
				position: new kakao.maps.LatLng(place.y, place.x)
			});

			// 마커에 클릭이벤트를 등록합니다
			kakao.maps.event.addListener(marker, 'click', function() {
				// 마커를 클릭하면 장소명이 인포윈도우에 표출됩니다
				infowindow.setContent('<div style="padding:5px;font-size:12px;">' + place.place_name + '</div>');
				infowindow.open(map, marker);
			});
		}

		

		// 지도에 확진자 동선 표시
		// 사용자 이동 데이터(좌표,시간)
		var gpsdata = [
			[37.262474090087416, 127.04188715342302],
			[37.26258509389554, 127.04194079759993],
			[37.26264486510906, 127.04185496691689],
			[37.262627787624325, 127.04169403438617],
			[37.262499706365354, 127.04164039020927],
			[37.26237162488855, 127.0415652883616],
			[37.262422857505406, 127.04140435583088],
			[37.26243993503662, 127.04125415213556],
			[37.262482628847685, 127.04117905028788],
			[37.262482628847685, 127.04116832145252],
			[37.26250824512272, 127.0411146772756],
			[37.26260217138997, 127.04083572755567],
			[37.262636326367186, 127.04068552386036],
			[37.26261071013571, 127.0405996931773],
			[37.26249116760702, 127.04055677783579],
			[37.26237162488855, 127.04049240482348],
			[37.26237162488855, 127.04034220112817],
			[37.26243993503662, 127.04019199743281],
			[37.26243993503662, 127.04000960723134],
			[37.26249116760702, 127.03989159004215],
			[37.26249116760702, 127.03980575935911],
			[37.26252532263456, 127.03974138634682],
			[37.26256801639724, 127.03959118265146],
			[37.262636326367186, 127.03948389429765],
			[37.26266194258993, 127.03940879245],
			[37.26269609754005, 127.03933369060233],
			[37.262713175009296, 127.03922640224852],
			[37.262764407393824, 127.0390761985532],
			[37.26281563974349, 127.03886162184558],
			[37.262883949488824, 127.03868996047947],
			[37.262994883059946, 127.03805699683464],
			[37.26308027002872, 127.03774586060861],
			[37.2631229634768, 127.03761711458402],
			[37.26319127294339, 127.03730597835798],
			[37.26326812101926, 127.03715577466265],
			[37.26327665968951, 127.0369519267904],
			[37.263327891690686, 127.03682318076582],
			[37.26343035558856, 127.03663006172896],
			[37.26347304883817, 127.0364369426921],
			[37.26350720342044, 127.03627601016139],
			[37.26373774644564, 127.03641548502134],
			[37.264070751791664, 127.03653350221055],
			[37.2641646761102, 127.0365549598813],
			[37.26430129309171, 127.03667297707051],
			[37.26440375566519, 127.03670516357663],
			[37.26433544729834, 127.03664079056436],
			[37.26474529656996, 127.03683390960123],
			[37.26518075899953, 127.03699484212858],
			[37.26559914207757, 127.0371450458239],
			[37.265710141279556, 127.03716650349465],
			[37.26582967869871, 127.03724160534232],
			[37.265923600823534, 127.03718796116542],
			[37.26600044611122, 127.03696265562243],
			[37.26608582967226, 127.03667297706714],
			[37.266179751477594, 127.03625455248726],
			[37.266128521416476, 127.03642621385337],
			[37.266205366495065, 127.03613653529808],
			[37.266256596503894, 127.03583612790742],
			[37.26631636480347, 127.03573956838898],
			[37.26636759473679, 127.03547134750444],
			[37.266384671373494, 127.03532114380913],
			[37.26644443957139, 127.03510656710151],
			[37.266478592806074, 127.03480615971081],
			[37.26649566941758, 127.03480615971081],
			[37.26655543752742, 127.03466668485086],
			[37.266478592806074, 127.0346023118386],
			[37.26636759473679, 127.03443065047249],
		];

		// 선을 구성하는 좌표 배열입니다. 이 좌표들을 이어서 선을 표시합니다
		var linePath = [];

		for(var i = 0;i<gpsdata.length;i++){
			linePath.push(new kakao.maps.LatLng(gpsdata[i][0], gpsdata[i][1]));
		}

		// 지도에 표시할 선을 생성합니다
		var polyline = new kakao.maps.Polyline({
			path: linePath, // 선을 구성하는 좌표배열 입니다
			strokeWeight: 2, // 선의 두께 입니다
			strokeColor: '#fb7c01', // 선의 색깔입니다
			strokeOpacity: 1, // 선의 불투명도 입니다 1에서 0 사이의 값이며 0에 가까울수록 투명합니다
			strokeStyle: 'solid' // 선의 스타일입니다
		});

		// 지도에 선을 표시합니다 
		polyline.setMap(map);  
		
		for(var i = 0;i<gpsdata.length;i++){
			var circle = new kakao.maps.Circle({
				center : new kakao.maps.LatLng(gpsdata[i][0], gpsdata[i][1]),  // 원의 중심좌표 입니다 
				radius: 3, // 미터 단위의 원의 반지름입니다 
				strokeWeight: 0, // 선의 두께입니다 
				strokeColor: '#fb7c01', // 선의 색깔입니다
				strokeOpacity: 1, // 선의 불투명도 입니다 1에서 0 사이의 값이며 0에 가까울수록 투명합니다
				strokeStyle: 'dashed', // 선의 스타일 입니다
				fillColor: '#fb0101', // 채우기 색깔입니다
				fillOpacity: 1  // 채우기 불투명도 입니다   
			}); 

			// 지도에 원을 표시합니다 
			circle.setMap(map);
		}
	</script>
</body>
</html>
