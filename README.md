import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Movie;
import android.net.Uri;
import android.os.SystemClock;
import android.util.AttributeSet;
import android.util.Log;
import android.view.View;
import java.io.FileNotFoundException;
import java.io.InputStream;

public class MyGifView extends View {
    
    private InputStream mInputStream;
    private Movie mMovie;
    private int mWidth, mHeight;
    private long mStart;
    private Context mContext;

    public MyGifView(Context mContext) {
        super(mContext);
        this.mContext = mContext;
    }

    public MyGifView(Context mContext, AttributeSet attributeSet) {
        this(mContext, attributeSet, 0);
    }

    public MyGifView(Context mContext, AttributeSet attributeSet, int defStyleAttr) {
        super(mContext, attributeSet, defStyleAttr);
        this.mContext = mContext;
        if (attributeSet.getAttributeName(1).equals("background")) {
            int id = Integer.parseInt(attributeSet.getAttributeValue(1).substring(1));
            setGifImageResource(id);
        }
    }

    private void init() {
        setFocusable(true);
        mMovie = Movie.decodeStream(mInputStream);
        mWidth = mMovie.width();
        mHeight = mMovie.height();

        requestLayout();
    }
    
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(mWidth, mHeight);
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        long now = SystemClock.uptimeMillis();
        if (mStart == 0) {
            mStart = now;
        }
        if (mMovie != null) {
            int duration = mMovie.duration();
            if (duration == 0) {
                duration = 1000;
            }
            int relTime = (int) ((now - mStart) % duration);
            mMovie.setTime(relTime);

            mMovie.draw(canvas, 0, 0);
            invalidate();
        }
    }

    public void setGifImageResource(int id) {
        mInputStream = mContext.getResources().openRawResource(id);
        init();
    }

    public void setGifImageUrl(Uri uri) {
        try {
            mInputStream = mContext.getContentResolver().openInputStream(uri);
            init();
        } catch (FileNotFoundException e) {
            Log.e("MyGifView.class : ", "File not found.");
        }
    }
}
