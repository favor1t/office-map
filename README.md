# office-map
office map ( konva.js )
/*Построение карты офиса*/

var Map_Office = (function(){
	var that = function(){return this;};
	
	var json_map_users = function(){
	var that = this;
	$.ajax({
			url: "/local/modules/map_office/api/?floor="+that.Floor,
		})
		.done(function(data) {
		
		
				that.data = $.parseJSON(data);
				that.AddToMap($.parseJSON(data),that.imageObj());
		});
		
	};
	
	var addUserToMap = function(data,imageObj){

		var that = this;
		
		that.stageObj = new Konva.Stage({
								container: 'container',
								width: that.width,
								height: that.height,
							});

	
		$.each(data.users, function (i, val){

			var img = 	new Konva.Image({
							x: parseInt(val.x),
							y: parseInt(val.y),
							id: parseInt(val.id),
							user_name: val.user_name,
							user_id: val.user_id,
							image: imageObj,
							width: 23,
							height: 23,
							draggable: that.draggable,
							stroke: val.stroke,
							strokeWidth:parseInt(val.strokeWidth),
							className: 'user',
						});

			// добавлялем стили для курсора при событии mouseover
			img.on('mouseover', function() {
				that.setTooltipUserFields(this);
				that.TooltipUserToggle('tooltip','show');
				document.body.style.cursor = 'pointer';
			});
			// добавлялем стили для курсова при событии mouseout
			img.on('mouseout', function() {
				that.setTooltipUserFields({});
				that.TooltipUserToggle('tooltip','hide');
				document.body.style.cursor = 'default';
			});
			if(that.draggable==true)
			{
				//добавляем событие click по элементам на слоям
				img.on('click', function(event) {
					that.SetEditFields(this);
					$('#edit').modal({show: true});
				});			
				//событие на drag
				img.on('dragmove', function() {
					that.setTooltipUserFields(this);
				});
			}
			imageObj.onload = function(){
				img.image(imageObj);
				that.layer.draw();
			}; 
			that.layerAdd(img);
		});
		that.stageObj.add(that.layer)	
	};
	
	return {
		/*ID Этажа*/
		Floor: 1,
		
		/*Ширина области*/
		width: 825,
		
		/*Высота области*/
		height: 550,
		
		/*JSON данные из  AddToMap: addUserToMap*/
		data: '',
		
		/*Получение json данных объектов для карты офиса*/
		build: json_map_users,
		
		/*Добавление объектов на карту*/
		AddToMap: addUserToMap,
		
		init:					function(){
									this.selectFloor();
									this.stage();
									this.build();
								},
		setTooltipUserFields: 	function(obj){
									if($.isEmptyObject(obj))
									{
										$('#userName').text('');
										$('#userID').text('');	
										$('#userPosition').text('');	
									}
									else
									{
										$('#userName').text(obj.getAttr('user_name'));
										$('#userID').text(obj.id());
										$('#userPosition').text(obj.getX()+' : '+obj.getY());
									}
								},
		SetEditFields: 			function(user)
									{
										$('#edit .userName').text(user.getAttr('user_name')+'[id:'+user.getAttr('user_id')+']');
										$('#edit .positionX').val(user.getX());
										$('#edit .positionY').val(user.getY());
										$('#edit .id').val(user.id());
										$('#edit .user_name').val(user.getAttr('user_name'));
										
									},
		TooltipUserToggle: 		function(css_class,toggle)
									{
										if(toggle=='show')
										{
											$("."+css_class).show();
										}
										else
										{
											$("."+css_class).hide();
										}

									},
		imageObj: 				function()
									{
										var imageObj = new Image();
										imageObj.src = 'user.png';
										return imageObj;
									},
		layer:					new Konva.Layer(),
		draw:					function(){this.layer.draw();},
		layerAdd:				function(img){
									this.layer.add(img);
								},
		stage:					function(w,h){
									return new Konva.Stage({
											container: 'container',
											width: that.width,
											height: that.height,
										});
								},
		stageObj:				{},
		selectFloor:			function(){
									var that = this;
									$( "#selectFloor").change(function() {
										that.Floor = $( "#selectFloor").val();
										that.rebuild();
									});
								},
		rebuild:				function(){
									this.stageObj.destroy();
									this.build();
								},
		draggable:				true,
								
	}
	
})();

$('.adduser').click(function(e){
	e.preventDefault();
	$('#add').modal({show: true});
});

function ajaxsend()
{
 $('.form').submit(function(e) {
		//предотвращаем стандартные действия при клике
        e.preventDefault();
		var that = this;
        // отправка запроса
        $.ajax({
            type        : $(this).attr('method'),
            url         : $(this).attr('action'), 
            data        : $(this).serializeArray(), 
            dataType    : 'json', 
            encode      : true
        })
        .done(function(data) {
			StatusAction(data,$(that).attr('name'));
        });

        
    });
}

function StatusAction(data,id)
{
	$('#'+id).modal('hide');
	$('#status .status').html(data.status);
	$('#status .text').html(data.title);
	$('#status').modal({show: true});
	
	setTimeout(function(){$('#status').modal('hide')}, 1000);
	Map_Office.rebuild();
} 
