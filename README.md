# openhsp_diff

掲示板で報告するときにOpenHSPの変更点を示すためのものです。

---

HSP3Dish.js での以下の件について  
  
１. 動作解像度(ENV.HSP_WX,WY)と表示解像度(ENV.HSP_SX,SY)が異なるときmtinfoやmousexの値がズレる  
    https://hsp.tv/play/pforum.php?mode=all&num=101997#102412  
２. 『HGIMG4を使用する』ON の場合、動作解像度と表示解像度が異なるとオートスケーリングされない  
３. 『HGIMG4を使用する』ON の場合、マウスクリックのmtinfoタッチ識別用IDが -1 にならず 0 になる  
    https://hsp.tv/play/pforum.php?mode=all&num=101997#102696  

---

１. 動作解像度(ENV.HSP_WX,WY)と表示解像度(ENV.HSP_SX,SY)が異なるときmtinfoやmousexの値がズレる  
  
HSP3本体ソース: hsp3dish/emscripten/hgiox.cpp (115行目,130行目) - case SDL_FINGERMOTION,SDL_FINGERUP 内  
<pre>
				bm = (Bmscr *)exinfo->HspFunc_getbmscr(0);
				x = event.tfinger.x * bm->sx;
				y = event.tfinger.y * bm->sy;
</pre>
の bm->sx(bm->sy) が HSP_WX(HSP_WY) の値になっているので、ここが HSP_SX(HSP_SY) の値になるように修正すると良いみたいです。  
<br>
<br>
２.『HGIMG4を使用する』ON の場合、動作解像度と表示解像度が異なるとオートスケーリングされない  
  
基本的には hsp3dish/gameplay/src/Game.cpp 内の glViewport()関数の引数を、HGIMG4版ではない通常版の hsp3dish/emscripten/hgiox.cpp 内の glViewport() でスケーリングされている方法と同じように記述すればうまく動作しました。  
<br>
<br>
３. 『HGIMG4を使用する』ON の場合、マウスクリックのmtinfoタッチ識別用IDが -1 にならず 0 になる  
  
HSP3本体ソース: hsp3dish/win32gp/hgiox.cpp (2036行目) - hgio_touch()関数内   
<pre>
        bm->setMTouchByPointId( 0, mouse_x, mouse_y, button!=0 );
        
        ↓ 下記に変更
        
        HSP3MTOUCH *mt;
		bool notice = false;
		if (button!=0) {
			mt = bm->getMTouchByPointId(-1);
			if (mt==NULL) {
				mt = bm->getMTouch(0);
				if (mt->flag == 0) notice=true;
			} else {
				notice=true;
			}
		} else {
			notice=true;
		}
		if (notice) {
	        bm->setMTouchByPointId( -1, mouse_x, mouse_y, button!=0 );
		}
</pre>
これは HGIMG4版ではない通常版で使用される hsp3dish/emscripten/hgiox.cpp 内の hgio_touch()関数と同じ記述になるようにしただけです。  
