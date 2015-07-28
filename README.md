# henan
手机app手势密码
import android.app.Activity;
import android.app.AlertDialog;
import android.content.Context;
import android.content.Intent;
import android.content.SharedPreferences;
import android.content.SharedPreferences.Editor;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Typeface;
import android.graphics.Paint.Cap;
import android.os.Handler;
import android.util.AttributeSet;

import android.view.MotionEvent;
import android.view.View;
import android.widget.Toast;

/**
 * 作用：手势密码的九宫格 
 * */
public class NinePointLineView extends View {

	private	Paint linePaint = new Paint();//画笔

	private	Paint whiteLinePaint = new Paint();

	private	Paint textPaint = new Paint();

	private	Bitmap defaultBitmap = BitmapFactory.decodeResource(getResources(),
			R.drawable.lock);

	private	int defaultBitmapRadius = defaultBitmap.getWidth() / 2;

	private	Bitmap selectedBitmap = BitmapFactory.decodeResource(getResources(),
			R.drawable.indicator_lock_area);

	private	int selectedBitmapDiameter = selectedBitmap.getWidth();

	private	int selectedBitmapRadius = selectedBitmapDiameter / 2;

	private	PointInfo[] points = new PointInfo[9];

	private	PointInfo startPoint = null;

	private	int width, height;

	private	int moveX, moveY;

	private boolean isUp = false;

	private Context context;

	StringBuffer lockString = new StringBuffer();
	

	public NinePointLineView(Context context) {

		super(context);

		this.context = context;

//		this.setBackgroundColor(Color.WHITE);//设置整个背景

		initPaint();
	}

	public NinePointLineView(Context context, AttributeSet attrs) {

		super(context, attrs);
		this.context = context;
//		this.setBackgroundColor(Color.WHITE);//设置整个九宫格的背景

		initPaint();
	}

	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

		width = getWidth();

		height = getHeight();

		if (width != 0 && height != 0) {

			initPoints(points);

		}

		super.onMeasure(widthMeasureSpec, heightMeasureSpec);

	}

	@Override
	protected void onLayout(boolean changed, int left, int top, int right,
			int bottom) {

		super.onLayout(changed, left, top, right, bottom);

	}

	private int startX = 0, startY = 0;

	@Override
	protected void onDraw(Canvas canvas) {

		// canvas.drawText("passwd:" + lockString, 0, 40, textPaint);

		if (moveX != 0 && moveY != 0 && startX != 0 && startY != 0) {
			// drawLine(canvas, startX, startY, moveX, moveY);
		}

		drawNinePoint(canvas);

		super.onDraw(canvas);
	}

	@Override
	public boolean onTouchEvent(MotionEvent event) {

		boolean flag = true;

		if (isUp) {

			finishDraw();

			flag = false;

		} else {
			handlingEvent(event);

			flag = true;

		}
		return flag;
	}

	private void handlingEvent(MotionEvent event) {

		switch (event.getAction()) {

		case MotionEvent.ACTION_MOVE:

			moveX = (int) event.getX();

			moveY = (int) event.getY();

			for (PointInfo temp : points) {

				if (temp.isInMyPlace(moveX, moveY)
						&& temp.isSelected() == false) {

					temp.setSelected(true);

					startX = temp.getCenterX();

					startY = temp.getCenterY();

					int len = lockString.length();

					if (len != 0) {

						int preId = lockString.charAt(len - 1) - 48;

						points[preId].setNextId(temp.getId());

					}

					lockString.append(temp.getId());

					break;
				}
			}

			invalidate(0, height - width, width, height);

			break;

		case MotionEvent.ACTION_DOWN:

			int downX = (int) event.getX();

			int downY = (int) event.getY();

			for (PointInfo temp : points) {

				if (temp.isInMyPlace(downX, downY)) {

					temp.setSelected(true);

					startPoint = temp;

					startX = temp.getCenterX();

					startY = temp.getCenterY();

					lockString.append(temp.getId());

					break;
				}
			}

			invalidate(0, height - width, width, height);

			break;

		case MotionEvent.ACTION_UP:

			startX = startY = moveX = moveY = 0;

			isUp = true;

			invalidate();
			Handler handler1=new Handler();
			handler1.post(savePwd);
			break;

		default:
			break;
		}
	}

	private void finishDraw() {

		for (PointInfo temp : points) {

			temp.setSelected(false);

			temp.setNextId(temp.getId());

		}

		lockString.delete(0, lockString.length());

		isUp = false;

		invalidate();
	}

	private void initPoints(PointInfo[] points) {

		int len = points.length;

		int seletedSpacing = (width - selectedBitmapDiameter * 3) / 4;

		int seletedX = seletedSpacing;

		int seletedY = height - width + seletedSpacing;

		int defaultX = seletedX + selectedBitmapRadius - defaultBitmapRadius;

		int defaultY = seletedY + selectedBitmapRadius - defaultBitmapRadius;

		for (int i = 0; i < len; i++) {

			if (i == 3 || i == 6) {

				seletedX = seletedSpacing;

				seletedY += selectedBitmapDiameter + seletedSpacing;

				defaultX = seletedX + selectedBitmapRadius
						- defaultBitmapRadius;

				defaultY += selectedBitmapDiameter + seletedSpacing;

			}

			points[i] = new PointInfo(i, defaultX, defaultY, seletedX, seletedY);

			seletedX += selectedBitmapDiameter + seletedSpacing;

			defaultX += selectedBitmapDiameter + seletedSpacing;

		}
	}

	private void initPaint() {

		initLinePaint(linePaint);

		initTextPaint(textPaint);

		initWhiteLinePaint(whiteLinePaint);

	}

	/**
	 * @param paint
	 */
	private void initTextPaint(Paint paint) {

		textPaint.setTextSize(30);

		textPaint.setAntiAlias(true);

		textPaint.setTypeface(Typeface.MONOSPACE);

	}

	/**
	 * @param paint
	 */
	private void initLinePaint(Paint paint) {

		paint.setARGB(3, 77, 115, 115);

		paint.setStrokeWidth(8);//设置两个点之间的线的背景的宽度

		paint.setAntiAlias(true);

		paint.setStrokeCap(Cap.ROUND);

	}

	/**
	 * @param paint
	 */
	private void initWhiteLinePaint(Paint paint) {

		paint.setARGB(180, 3, 77, 115);

		paint.setStrokeWidth(7);//设置两个点之间的线的宽度

		paint.setAntiAlias(true);

		paint.setStrokeCap(Cap.ROUND);

	}
	/**
	 * 
	 * @param canvas
	 */
	private void drawNinePoint(Canvas canvas) {

		if (startPoint != null) {

			drawEachLine(canvas, startPoint);

		}

		for (PointInfo pointInfo : points) {

			if (pointInfo != null) {

				if (pointInfo.isSelected()) {
                    
					canvas.drawBitmap(selectedBitmap, pointInfo.getSeletedX(),
							pointInfo.getSeletedY(), null);

				}else{

				canvas.drawBitmap(defaultBitmap, pointInfo.getDefaultX(),
						pointInfo.getDefaultY(), null);
				}
			}
		}

	}

	/**
	 * @param canvas
	 * @param point
	 */
	private void drawEachLine(Canvas canvas, PointInfo point) {
		if (point.hasNextId()) {
			int n = point.getNextId();
			drawLine(canvas, point.getCenterX(), point.getCenterY(),
					points[n].getCenterX(), points[n].getCenterY());
			drawEachLine(canvas, points[n]);
		}
	}

	/**
	 * 
	 * @param canvas
	 * @param startX
	 * @param startY
	 * @param stopX
	 * @param stopY
	 */
	private void drawLine(Canvas canvas, float startX, float startY,
			float stopX, float stopY) {

		//canvas.drawLine(startX, startY, stopX, stopY, linePaint);

		canvas.drawLine(startX, startY, stopX, stopY, whiteLinePaint);

	}

	/**
	 * 
	 * 
	 */
	private class PointInfo {

		private int id;

		private int nextId;

		private boolean selected;

		private int defaultX;

		private int defaultY;

		private int seletedX;

		private int seletedY;

		public PointInfo(int id, int defaultX, int defaultY, int seletedX,
				int seletedY) {
			this.id = id;
			this.nextId = id;
			this.defaultX = defaultX;
			this.defaultY = defaultY;
			this.seletedX = seletedX;
			this.seletedY = seletedY;
		}

		public boolean isSelected() {
			return selected;
		}

		public void setSelected(boolean selected) {
			this.selected = selected;
		}

		public int getId() {
			return id;
		}

		public int getDefaultX() {
			return defaultX;
		}

		public int getDefaultY() {
			return defaultY;
		}

		public int getSeletedX() {
			return seletedX;
		}

		public int getSeletedY() {
			return seletedY;
		}

		public int getCenterX() {
			return seletedX + selectedBitmapRadius;
		}

		public int getCenterY() {
			return seletedY + selectedBitmapRadius;
		}

		public boolean hasNextId() {
			return nextId != id;
		}

		public int getNextId() {
			return nextId;
		}

		public void setNextId(int nextId) {
			this.nextId = nextId;
		}

		/**
		 * @param x
		 * @param y
		 */
		public boolean isInMyPlace(int x, int y) {
			boolean inX = x > seletedX
					&& x < (seletedX + selectedBitmapDiameter);
			boolean inY = y > seletedY
					&& y < (seletedY + selectedBitmapDiameter);

			return (inX && inY);
		}

	}

	
	
	/**
	 * 作用：保存密码并且判断界面的跳转
	 * */
	Thread savePwd=new Thread(){
		public void run(){
			Intent intent = new Intent();
			SharedPreferences shareDate = getContext().getSharedPreferences("GUE_PWD", 0);
			boolean isSetFirst = shareDate.getBoolean("IS_SET_FIRST", false);
			boolean isSet=shareDate.getBoolean("IS_SET", false);
			SharedPreferences changepwd = getContext().getSharedPreferences("changePWD", 0);
			String change=changepwd.getString("changeok","");
			if(isSet){
		         String pwd1 = this.getPwd();
				String GUE_PWD = shareDate.getString("GUE_PWD", "NO HAVE PWD");
	             if(pwd1.equals(GUE_PWD)){//密码和第一次设置密码一样  
					shareDate.edit().clear().commit();
					shareDate.edit().putBoolean("IS_SET", true).commit();
					shareDate.edit().putString("GUE_PWD", pwd1).commit();	
                    if(getReturnValue().equals("3")){
                    	Toast.makeText(getContext(), getReturnMemo(), 0).show();
                    	SharedPreferences p1= getContext().getSharedPreferences("zlperson", 0);
						SharedPreferences p2= getContext().getSharedPreferences("zlreturn", 0);
						p1.edit().clear().commit();
						p2.edit().clear().commit();
						shareDate.edit().clear().commit();
						SharedPreferences zlinsert= getContext().getSharedPreferences("zlinsert", 0);
						zlinsert.edit().clear().commit();
						intent.setClass(context,LoginActivity.class);
					}
                    else if(getReturnValue().equals("1")){
                    	if(change.equals("ok")){
                    	SharedPreferences p1=getContext().getSharedPreferences("GUE_PWD", 0);
                    	SharedPreferences p2 = getContext().getSharedPreferences("changePWD", 0);
            			p1.edit().clear().commit();
            			p2.edit().clear().commit();
                    	intent.setClass(context,GestureActivity.class);
                			
        				}
                    	else{
				      intent.setClass(context,MainActivity.class);
                    	}
                    }
                    else{
                    	Toast.makeText(getContext(), getReturnMemo(), 0).show();
                    	SharedPreferences p1= getContext().getSharedPreferences("zlperson", 0);
						SharedPreferences p2= getContext().getSharedPreferences("zlreturn", 0);
						p1.edit().clear().commit();
						p2.edit().clear().commit();
						shareDate.edit().clear().commit();
						SharedPreferences zlinsert= getContext().getSharedPreferences("zlinsert", 0);
						zlinsert.edit().clear().commit();
						intent.setClass(context,LoginActivity.class);
                    }
                    
		       }else{//密码和第一次设置密码不一样 
					int time=shareDate.getInt("time",5);
					shareDate.edit().putBoolean("RESECOND_ERROR", true).commit();
					intent.setClass(context, GestureActivity.class);
	                time--;
					shareDate.edit().putInt("time", time--).commit();
					if(time==-1){
						SharedPreferences p1= getContext().getSharedPreferences("zlperson", 0);
						SharedPreferences p2= getContext().getSharedPreferences("zlreturn", 0);
						p1.edit().clear().commit();
						p2.edit().clear().commit();
						SharedPreferences zlinsert= getContext().getSharedPreferences("zlinsert", 0);
						zlinsert.edit().clear().commit();
						shareDate.edit().clear().commit();
						intent.setClass(context,LoginActivity.class);
					}
					
				}
			}
			else{
			if(isSetFirst){//如果第一次已经设置密码，验证第二次和第一次是否一致
				
				String pwd = this.getPwd();
				
				String first_pwd = shareDate.getString("FIRST_PWD", "NO HAVE PWD");
				
				if(pwd.equals(first_pwd)){//第二次密码和第一次密码一样   设置成功
					
					shareDate.edit().clear().commit();
					
					shareDate.edit().putBoolean("IS_SET", true).commit();
					
					shareDate.edit().putString("GUE_PWD", pwd).commit();
					intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
					intent.setClass(context, MainActivity.class);
					
				}else{//第二次输入的密码和第一次输入的密码不一致
					
					
					shareDate.edit().putBoolean("SECOND_ERROR", true).commit();
					
					intent.setClass(context, GestureActivity.class);
					
				}
				
			}else{//第一次设置手势密码
				
				shareDate.edit().clear().commit();
				
				shareDate.edit().putString("FIRST_PWD", this.getPwd()).commit();
				
				shareDate.edit().putBoolean("IS_SET_FIRST", true).commit();
				
				intent.setClass(context, GestureActivity.class);
					}
			}
			
			((Activity) context).startActivity(intent);
			((Activity) context).finish();
			
			
		}

		public String getPwd() {//获取本次的密码

			return lockString.toString();

		}
		
	};


	/*取得返回值*/	
	public String getReturnValue(){
		SharedPreferences preferences=getContext().getSharedPreferences("zlreturn",Context.MODE_PRIVATE);
		String returnValue=preferences.getString("returnValue", "");
		return returnValue;
		
	}
	/*取得返回结果*/	
	public String getReturnMemo(){	
		SharedPreferences preferences=getContext().getSharedPreferences("zlreturn",Context.MODE_PRIVATE);
		String returnMemo=preferences.getString("returnMemo", "");
		return returnMemo;
		
	}

}
