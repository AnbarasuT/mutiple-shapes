mutiple-shapes
==============

canvas



var _this;
window.QuadriAngles = function (canvas, details){
    _this = this;
    this.isTouchDevice = ('ontouchstart' in window) || ('onmsgesturechange' in window);
    
    this.canvas = canvas;
    this.context = this.canvas[0].getContext('2d');
    this.canvas.parent().append('<div id="dragArea"/>');
    $('#dragArea').css({'position':'absolute', 'left':'10px','top':'10px','width':(this.canvas.width()-20)+'px', 'height':(this.canvas.height()-40)+'px'})
    if (this.isTouchDevice) {
        $('#dragArea')[0].addEventListener('touchstart', function (event) { _this.onTouchStart(event);});
        $('#dragArea')[0].addEventListener('touchmove', function (event) { _this.onTouchMove(event);});
        window.addEventListener('touchend', function (event) { _this.onTouchEnd(event)});
    }
    else {
        $('#dragArea')[0].addEventListener('mousedown', function (event) { _this.onTouchStart(event);});
        $('#dragArea')[0].addEventListener('mousemove', function (event) { _this.onTouchMove(event);});
        window.addEventListener('mouseup', function (event) { _this.onTouchEnd(event)});
    }
    
    this.quadLabels={
        nodes: ["A", "B", "C", "D"],
        edges: ["k", "l", "m", "n"],
    }
    
    this.quadStyles={
        lineWidth:2,
        strokeColor:"#000",
        fillColor:"rgba(153, 204, 255, 0.8)",
    }
    this.pointsStyle={
        lineWidth:1,
        strokeColor:"#000",
        fillColor:"rgba(255, 0, 0, 0.6)",
    }
    this.pixelPerUnit = 30;
    this.alignX = 300;
    this.alignY = 300;
    this.selectedShape = "square";
    this.shapes = [
                   {
                    name:'square',
                    points: [{x:this.alignX, y:this.alignY}, {x:this.alignX, y:this.alignY-100}, {x:this.alignX+100, y:this.alignY-100}, {x:this.alignX+100, y:this.alignY}]
                   },
                   {
                    name:'rectangle',
                    points: [{x:this.alignX-50, y:this.alignY}, {x:this.alignX-50, y:this.alignY-100}, {x:this.alignX+150, y:this.alignY-100}, {x:this.alignX+150, y:this.alignY}]
                   },
                   {
                    name:'rhombus',
                    points: [{x:this.alignX, y:this.alignY}, {x:this.alignX+30, y:this.alignY-100}, {x:this.alignX+135, y:this.alignY-100}, {x:this.alignX+105, y:this.alignY}]
                   },
                   {
                    name:'parallelogram',
                    points: [{x:this.alignX-20, y:this.alignY}, {x:this.alignX-70, y:this.alignY-100}, {x:this.alignX+100, y:this.alignY-100}, {x:this.alignX+150, y:this.alignY}]
                   },
                   {
                    name:'freeform',
                    points: [{x:this.alignX-50, y:this.alignY}, {x:this.alignX-70, y:this.alignY-100}, {x:this.alignX+100, y:this.alignY-70}, {x:this.alignX+80, y:this.alignY}]
                   },
                ];
    
    this.shapeTouchPoint = -1;
    
    this.init();
}
QuadriAngles.prototype = {
    init:function(){
        console.log("Activity Initialized...");
        this.clearAll();
        
        this.drawShape();
        this.showAngle();
    },
    clearAll:function (){
        this.context.clearRect(0, 0, this.canvas.width(), this.canvas.height());
    },
    reset:function(){
        this.selectedShape = "square";
        this.shapes = [{name:'square',points: [{x:this.alignX, y:this.alignY}, {x:this.alignX, y:this.alignY-100}, {x:this.alignX+100, y:this.alignY-100}, {x:this.alignX+100, y:this.alignY}]},
                   {name:'rectangle',points: [{x:this.alignX-50, y:this.alignY}, {x:this.alignX-50, y:this.alignY-100}, {x:this.alignX+150, y:this.alignY-100}, {x:this.alignX+150, y:this.alignY}]},
                   {name:'rhombus',points: [{x:this.alignX, y:this.alignY}, {x:this.alignX+30, y:this.alignY-100}, {x:this.alignX+135, y:this.alignY-100}, {x:this.alignX+105, y:this.alignY}]},
                   {name:'parallelogram',points: [{x:this.alignX-20, y:this.alignY}, {x:this.alignX-70, y:this.alignY-100}, {x:this.alignX+100, y:this.alignY-100}, {x:this.alignX+150, y:this.alignY}]},
                   {name:'freeform',points: [{x:this.alignX-50, y:this.alignY}, {x:this.alignX-70, y:this.alignY-100}, {x:this.alignX+100, y:this.alignY-70}, {x:this.alignX+80, y:this.alignY}]},
                ];
        
        this.init();
    },
    drawShape:function(){
        for (var shape=0; shape<this.shapes.length;shape++) {
            if (this.shapes[shape].name == this.selectedShape) {
                var points = this.shapes[shape].points;
                this.drawQuad(points[0].x, points[0].y, points[1].x, points[1].y, points[2].x, points[2].y, points[3].x, points[3].y, this.quadStyles, this.quadLabels);
                break;
            }
        }
    },
    drawQuad:function(startX, startY, x1, y1, x2, y2, x3, y3, styles, labels){
        this.context.beginPath();
        this.context.lineWidth=styles.lineWidth;
        this.context.strokeStyle=styles.strokeColor;
        this.context.fillStyle=styles.fillColor;
        this.context.moveTo(startX,startY);
        this.context.lineTo(x1,y1);
        this.context.lineTo(x2,y2);
        this.context.lineTo(x3,y3);
        this.context.lineTo(startX,startY);
        this.context.fill();
        this.context.stroke();
        
        var midX = (startX+x1+x2+x3)/4;
        var midY = (startY+y1+y2+y3)/4;
        
        this.context.beginPath();
        this.context.fillStyle="#000";
        this.context.font="italic 16px Verdana";
        
        pointAng = this.FindAngleFromPoints({x:midX, y:midY}, {x:startX, y:startY});
        dist = Math.sqrt(Math.pow(midY - startY, 2)+ Math.pow(midX - startX, 2))+25;
        this.context.fillText(labels.nodes[0], midX+dist * Math.cos(pointAng.angle * (Math.PI / 180)),midY+dist * Math.sin(pointAng.angle * (Math.PI / 180)));
        
        pointAng = this.FindAngleFromPoints({x:midX, y:midY}, {x:x1, y:y1});
        dist = Math.sqrt(Math.pow(midY - y1, 2)+ Math.pow(midX - x1, 2))+25;
        this.context.fillText(labels.nodes[1], midX+(dist-5) * Math.cos(pointAng.angle * (Math.PI / 180)),midY+(dist-10) * Math.sin(pointAng.angle * (Math.PI / 180)));
        
        pointAng = this.FindAngleFromPoints({x:midX, y:midY}, {x:x2, y:y2});
        dist = Math.sqrt(Math.pow(midY - y2, 2)+ Math.pow(midX - x2, 2))+25;
        this.context.fillText(labels.nodes[2], midX+(dist-15) * Math.cos(pointAng.angle * (Math.PI / 180)),midY+(dist-10) * Math.sin(pointAng.angle * (Math.PI / 180)));
        
        pointAng = this.FindAngleFromPoints({x:midX, y:midY}, {x:x3, y:y3});
        dist = Math.sqrt(Math.pow(midY - y3, 2)+ Math.pow(midX - x3, 2))+25;
        this.context.fillText(labels.nodes[3], midX+(dist-15) * Math.cos(pointAng.angle * (Math.PI / 180)),midY+dist * Math.sin(pointAng.angle * (Math.PI / 180)));
        this.context.fill();
        
        this.drawPoint(startX, startY, 6, 0, 2, this.pointsStyle);
        this.drawPoint(x1, y1, 6, 0, 2, this.pointsStyle);
        this.drawPoint(x2, y2, 6, 0, 2, this.pointsStyle);
        this.drawPoint(x3, y3, 6, 0, 2, this.pointsStyle);
    },
    drawLine:function(point1, point2){
        this.context.beginPath();
        this.context.lineWidth=this.lineStyles.lineWidth;
        this.context.strokeStyle=this.lineStyles.strokeColor;
        this.context.moveTo(point1.x,point1.y);
        this.context.lineTo(point2.x,point2.y);
        this.context.stroke();
    },
    drawHandle:function(point1, point2){
        this.context.beginPath();
        this.context.lineWidth=this.lineStyles.lineWidth;
        this.context.strokeStyle=this.lineStyles.strokeColor;
        this.context.moveTo(point1.x,point1.y);
        this.context.lineTo(point2.x,point2.y);
        this.context.stroke();
        this.drawPoint(point1.x, point1.y, 10, 0, 360, this.pointsStyle);
    },
    drawPoint:function(x, y, radius, sAngle, eAngle, styles){
        this.context.beginPath();
        this.context.lineWidth=styles.lineWidth;
        this.context.strokeStyle=styles.strokeColor;
        this.context.fillStyle=styles.fillColor;
        this.context.arc(x,y,radius,sAngle,eAngle*Math.PI);
        this.context.fill();
        this.context.stroke();
    },
    showAngle:function(){
        var points = [];
        for (var shape=0; shape<this.shapes.length;shape++) {
            if (this.shapes[shape].name == this.selectedShape) {
                points = this.shapes[shape].points;
                break;
            }
        }
        
        var angleA =  this.FindAngleFrom3Points(points[1], points[0], points[3]);
        var angleB =  this.FindAngleFrom3Points(points[0], points[1], points[2]);
        var angleC =  this.FindAngleFrom3Points(points[1], points[2], points[3]);
        var angleD =  this.FindAngleFrom3Points(points[2], points[3], points[0]);
        
        var abLen =  this.FindDistanceFromPoints(points[0], points[1]);
        var bcLen =  this.FindDistanceFromPoints(points[1], points[2]);
        var cdLen =  this.FindDistanceFromPoints(points[2], points[3]);
        var daLen =  this.FindDistanceFromPoints(points[3], points[0]);
        
        abLen = (abLen/this.pixelPerUnit).toFixed(1);
        bcLen = (bcLen/this.pixelPerUnit).toFixed(1);
        cdLen = (cdLen/this.pixelPerUnit).toFixed(1);
        daLen = (daLen/this.pixelPerUnit).toFixed(1);
        
        $('.aangle').val(Math.round(angleA));
        $('.bangle').val(Math.round(angleB));
        $('.cangle').val(Math.round(angleC));
        $('.dangle').val(Math.round(angleD));
        
        if (_this.selectedShape == "square" || _this.selectedShape=="rhombus") {
            $('.ablen').val(abLen);
            $('.bclen').val(abLen);
            $('.cdlen').val(abLen);
            $('.dalen').val(abLen);
        }
        else {
            $('.ablen').val(abLen);
            $('.bclen').val(bcLen);
            $('.cdlen').val(cdLen);
            $('.dalen').val(daLen);
        }
        
        this.displaySymbols(points);
    },
    displaySymbols:function(points){
        this.context.beginPath();
        this.context.lineWidth="1";
        this.context.strokeStyle="#000";
        
        if (this.selectedShape=="square" || this.selectedShape=="rectangle") {
            this.context.moveTo(points[0].x, points[0].y-15);
            this.context.lineTo(points[0].x+15, points[0].y-15);
            this.context.lineTo(points[0].x+15, points[0].y);
            this.context.moveTo(points[1].x, points[1].y+15);
            this.context.lineTo(points[1].x+15, points[1].y+15);
            this.context.lineTo(points[1].x+15, points[1].y);
            this.context.moveTo(points[2].x, points[2].y+15);
            this.context.lineTo(points[2].x-15, points[2].y+15);
            this.context.lineTo(points[2].x-15, points[2].y);
            this.context.moveTo(points[3].x, points[3].y-15);
            this.context.lineTo(points[3].x-15, points[3].y-15);
            this.context.lineTo(points[3].x-15, points[3].y);
            this.context.stroke();
        }
        else if (this.selectedShape=="rhombus" || this.selectedShape=="parallelogram") {
            var ang1 = this.FindAngleFromPoints(points[0], points[1]);
            var ang2 = this.FindAngleFromPoints(points[0], points[3]);
            var ang3 = ang1.angle - ang2.angle;
            this.context.arc(points[0].x, points[0].y, 15, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
            this.context.stroke();
            this.context.beginPath();
            var ang1 = this.FindAngleFromPoints(points[1], points[2]);
            var ang2 = this.FindAngleFromPoints(points[1], points[0]);
            var ang3 = ang1.angle - ang2.angle;
            this.context.arc(points[1].x, points[1].y, 15, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
            this.context.stroke();
            this.context.beginPath();
            this.context.arc(points[1].x, points[1].y, 17, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
            this.context.stroke();
            this.context.beginPath();
            var ang1 = this.FindAngleFromPoints(points[2], points[3]);
            var ang2 = this.FindAngleFromPoints(points[2], points[1]);
            var ang3 = ang1.angle - ang2.angle;
            this.context.arc(points[2].x, points[2].y, 15, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
            this.context.stroke();
            
            this.context.beginPath();
            var ang1 = this.FindAngleFromPoints(points[3], points[0]);
            var ang2 = this.FindAngleFromPoints(points[3], points[2]);
            var ang3 = ang1.angle - ang2.angle;
            this.context.arc(points[3].x, points[3].y, 15, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
            this.context.stroke();
            this.context.beginPath();
            this.context.arc(points[3].x, points[3].y, 17, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
            this.context.stroke();
        }
        else {
            var ang1 = this.FindAngleFromPoints(points[0], points[1]);
            var ang2 = this.FindAngleFromPoints(points[0], points[3]);
            var ang3 = ang1.angle - ang2.angle;
            this.context.arc(points[0].x, points[0].y, 15, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
            this.context.stroke();
            this.context.beginPath();
            var ang1 = this.FindAngleFromPoints(points[1], points[2]);
            var ang2 = this.FindAngleFromPoints(points[1], points[0]);
            var ang3 = ang1.angle - ang2.angle;
            this.context.arc(points[1].x, points[1].y, 15, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
            this.context.stroke();
            this.context.beginPath();
            var ang1 = this.FindAngleFromPoints(points[2], points[3]);
            var ang2 = this.FindAngleFromPoints(points[2], points[1]);
            var ang3 = ang1.angle - ang2.angle;
            this.context.arc(points[2].x, points[2].y, 15, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
            this.context.stroke();
            this.context.beginPath();
            var ang1 = this.FindAngleFromPoints(points[3], points[0]);
            var ang2 = this.FindAngleFromPoints(points[3], points[2]);
            var ang3 = ang1.angle - ang2.angle;
            this.context.arc(points[3].x, points[3].y, 15, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
            this.context.stroke();
        }
        this.context.beginPath();
        var midX = (points[0].x + points[1].x) /2;
        var midY = (points[0].y + points[1].y) /2;
        this.context.moveTo(midX, midY);
        this.context.lineTo(midX+8, midY);
        this.context.moveTo(midX, midY);
        this.context.lineTo(midX-8, midY);
        
        var midX = (points[1].x + points[2].x) /2;
        var midY = (points[1].y + points[2].y) /2;
        this.context.moveTo(midX, midY);
        this.context.lineTo(midX, midY-8);
        if (this.selectedShape=="rectangle" || this.selectedShape=="parallelogram") {
        this.context.moveTo(midX+4, midY);
        this.context.lineTo(midX+4, midY-8);
        }
        this.context.moveTo(midX, midY);
        this.context.lineTo(midX, midY+8);
        
        if (this.selectedShape=="rectangle" || this.selectedShape=="parallelogram") {
        this.context.moveTo(midX+4, midY);
        this.context.lineTo(midX+4, midY+8);
        }
        var midX = (points[2].x + points[3].x) /2;
        var midY = (points[2].y + points[3].y) /2;
        this.context.moveTo(midX, midY);
        this.context.lineTo(midX-8, midY);
        this.context.moveTo(midX, midY);
        this.context.lineTo(midX+8, midY);
        
        var midX = (points[3].x + points[0].x) /2;
        var midY = (points[3].y + points[0].y) /2;
        this.context.moveTo(midX, midY);
        this.context.lineTo(midX, midY-8);
        if (this.selectedShape=="rectangle" || this.selectedShape=="parallelogram") {
        this.context.moveTo(midX+4, midY);
        this.context.lineTo(midX+4, midY-8);
        }
        this.context.moveTo(midX, midY);
        this.context.lineTo(midX, midY+8);
        if (this.selectedShape=="rectangle" || this.selectedShape=="parallelogram") {
        this.context.moveTo(midX+4, midY);
        this.context.lineTo(midX+4, midY+8);
        }
        this.context.stroke();
    },
    onTouchStart:function(event){
        if (!event)event=window.event;
        var moX, moY;
        
        moX = _this.touchCoordsFromEvent(event, true)[0].x - _this.canvas.offset().left;
        moY = _this.touchCoordsFromEvent(event, true)[0].y - _this.canvas.offset().top;
        
        var points = [];
        for (var shape=0; shape<_this.shapes.length;shape++) {
            if (_this.shapes[shape].name == _this.selectedShape) {
                points = _this.shapes[shape].points;
                break;
            }
        }
        
        var midX = midY = 0;
        for(var point =0; point < points.length; point++)
        {
            midX += points[point].x;
            midY += points[point].y;
        }
        midX = midX / points.length;
        midY = midY / points.length;
        this.pointsAng = [];
        this.pointsDist = [];
        this.shapeMidX = midX;
        this.shapeMidY = midY;
        for(var point =0; point < points.length; point++)
        {
            var ang = _this.FindAngleFromPoints({x:midX, y:midY}, {x:points[point].x, y:points[point].y});
            var dist = _this.FindDistanceFromPoints({x:midX, y:midY}, {x:points[point].x, y:points[point].y});
            this.pointsAng.push(ang.angle);
            this.pointsDist.push(dist);
        }
        for(var point =0; point < points.length; point++)
        {
            if ((moX >= points[point].x-15 && moX <= points[point].x+15) && (moY >= points[point].y-15 && moY <= points[point].y+15)) {
                _this.shapeTouchPoint = point;
            }
        }
    },
    onTouchMove:function(event, canvas){
        if (!event)event=window.event;
        var moX, moY;
        moX = _this.touchCoordsFromEvent(event, false)[0].x - _this.canvas.offset().left;
        moY = _this.touchCoordsFromEvent(event, false)[0].y - _this.canvas.offset().top;
        
        if (_this.shapeTouchPoint!=-1) {
            var points = [];
            for (var shape=0; shape<_this.shapes.length;shape++) {
                if (_this.shapes[shape].name == _this.selectedShape) {
                    points = _this.shapes[shape].points;
                    break;
                }
            }
            
            if (_this.selectedShape == "square") {
                var dist = _this.FindDistanceFromPoints({x:this.shapeMidX, y:this.shapeMidY}, {x:moX, y:moY});
                for(var point =0; point < points.length; point++)
                {
                    points[point] = {x:this.shapeMidX+dist * Math.cos(this.pointsAng[point] * (Math.PI / 180)),y:this.shapeMidY+dist * Math.sin(this.pointsAng[point] * (Math.PI / 180))}
                }
                _this.clearAll();
                _this.drawShape();
                _this.showAngle();
                resetEnable();
            }
            else if (_this.selectedShape == "rectangle") {
                if (_this.shapeTouchPoint==0) {
                    if (moX < points[2].x-20) {
                        points[0].x = moX;
                        points[1].x = moX;
                    }
                    if (moY > points[2].y+20) {
                        points[0].y = moY;
                        points[3].y = moY;
                    }
                }
                else if (_this.shapeTouchPoint==1) {
                    if (moX < points[3].x-20) {
                        points[1].x = moX;
                        points[0].x = moX;
                    }
                    if (moY < points[3].y-20) {
                        points[1].y = moY;
                        points[2].y = moY;
                    }
                }
                else if (_this.shapeTouchPoint==2) {
                    if (moX > points[0].x+20) {
                        points[2].x = moX;
                        points[3].x = moX;
                    }
                    if (moY < points[0].y-20) {
                        points[2].y = moY;
                        points[1].y = moY;
                    }
                }
                else if (_this.shapeTouchPoint==3) {
                    if (moX > points[1].x+20) {
                        points[3].x = moX;
                        points[2].x = moX;
                    }
                    if (moY > points[1].y+20) {
                        points[3].y = moY;
                        points[0].y = moY;
                    }
                }
                _this.clearAll();
                _this.drawShape();
                _this.showAngle();
                resetEnable();
            }
            else if (_this.selectedShape == "rhombus") {
                
                var dist = _this.FindDistanceFromPoints({x:this.shapeMidX, y:this.shapeMidY}, {x:moX, y:moY});
                for(var point =0; point < points.length; point++)
                {
                    points[point] = {x:this.shapeMidX+(dist * this.pointsDist[point] / this.pointsDist[0]) * Math.cos(this.pointsAng[point] * (Math.PI / 180)),y:this.shapeMidY+(dist * this.pointsDist[point] / this.pointsDist[0]) * Math.sin(this.pointsAng[point] * (Math.PI / 180))}
                }
                
                _this.clearAll();
                _this.drawShape();
                _this.showAngle();
                resetEnable();
            }
            else if (_this.selectedShape == "parallelogram") {
                if (_this.shapeTouchPoint==0) {
                    if (moX < points[2].x-20) {
                        points[0].x = moX;
                        points[1].x = moX-50;
                    }
                    if (moY > points[2].y+20) {
                        points[0].y = moY;
                        points[3].y = moY;
                    }
                }
                else if (_this.shapeTouchPoint==1) {
                    if (moX < points[3].x-20) {
                        points[1].x = moX;
                        points[0].x = moX+50;
                    }
                    if (moY < points[3].y-20) {
                        points[1].y = moY;
                        points[2].y = moY;
                    }
                }
                else if (_this.shapeTouchPoint==2) {
                    if (moX > points[0].x+20) {
                        points[2].x = moX;
                        points[3].x = moX+50;
                    }
                    if (moY < points[0].y-20) {
                        points[2].y = moY;
                        points[1].y = moY;
                    }
                }
                else if (_this.shapeTouchPoint==3) {
                    if (moX > points[1].x+20) {
                        points[3].x = moX;
                        points[2].x = moX-50;
                    }
                    if (moY > points[1].y+20) {
                        points[3].y = moY;
                        points[0].y = moY;
                    }
                }
                _this.clearAll();
                _this.drawShape();
                _this.showAngle();
                resetEnable();
            }
            else if (_this.selectedShape == "freeform") {
                
                points[_this.shapeTouchPoint] = {x:moX,y:moY};
                
                _this.clearAll();
                _this.drawShape();
                _this.showAngle();
                resetEnable();
            }
        }
    },
    onTouchEnd:function(event){
        if (!event)event=window.event;
        console.log(event);
        var moX, moY;
        moX = _this.touchCoordsFromEvent(event, false)[0].x - _this.canvas.offset().left;
        moY = _this.touchCoordsFromEvent(event, false)[0].y - _this.canvas.offset().top;
        
        _this.shapeTouchPoint = -1;
    },
    executeFunction:function(funcObj){
        funcObj.executefunc();
    },
    touchCoordsFromEvent:function(event, isDown){
        var touches = [];
        if (isDown) {
            if (this.isTouchDevice)
                for (var tInd = 0; tInd < event.touches.length; tInd++)
                    touches[tInd] = { x: event.touches[tInd].pageX, y: event.touches[tInd].pageY };
            else touches[0] = { x: event.pageX, y: event.pageY };
        }
        else{
            if (this.isTouchDevice)
                for (var tInd = 0; tInd < event.changedTouches.length; tInd++)
                    touches[tInd] = { x: event.changedTouches[tInd].pageX, y: event.changedTouches[tInd].pageY };
            else touches[0] = { x: event.pageX, y: event.pageY };
        }
        return touches;
    },
    FindDistanceFromPoints:function(p1, p2){
        return Math.sqrt(Math.pow(p2.y - p1.y, 2)+ Math.pow(p2.x - p1.x, 2));
    },
    FindAngleFromPoints:function (p1, p2){
        var angleDeg = Math.atan2(p2.y - p1.y, p2.x - p1.x) * 180 / Math.PI;
        var angleRadians = (angleDeg) * (Math.PI / 180)
        return {angle:angleDeg, radian:angleRadians};
    },
    FindAngleFrom3Points:function(A, B, C){
        /*
        * Calculates the angle ABC (in degrees) 
        *
        * A first point
        * C second point
        * B center point
        */
        var AB = Math.sqrt(Math.pow(B.x-A.x,2)+ Math.pow(B.y-A.y,2));    
        var BC = Math.sqrt(Math.pow(B.x-C.x,2)+ Math.pow(B.y-C.y,2)); 
        var AC = Math.sqrt(Math.pow(C.x-A.x,2)+ Math.pow(C.y-A.y,2));
        return Math.acos((BC*BC+AB*AB-AC*AC)/(2*BC*AB)) * 57.2957795;  
    },
    PointInShape: function(nvert, vertx, verty, moX, moY ) {
        //nvert - Number of vertices in the polygon. Whether to repeat the first vertex at the end is discussed below.
        //vertx, verty - Arrays containing the x- and y-coordinates of the polygon's vertices.
        //moX, moY - X- and y-coordinate of the mouse.
        var i, j, c = false;
        for( i = 0, j = nvert-1; i < nvert; j = i++ ) {
            if( ( ( verty[i] > moY ) != ( verty[j] > moY ) ) &&
                ( moX < ( vertx[j] - vertx[i] ) * ( moY - verty[i] ) / ( verty[j] - verty[i] ) + vertx[i] ) ) {
                    c = !c;
            }
        }
        return c;
    },
};
