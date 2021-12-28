###created a Material Function with a custom node to remove repetitive patterns while sampling a texture


Original Code from 
https://www.iquilezles.org/www/articles/texturerepetition/texturerepetition.htm


```
// input float2 uv

// input Texture Object inTex

// input float blendRatio


 struct Functions
 {
	float4 hash4(float2 p)
	{
		return frac(sin(float4( 1.0+dot(p,float2(37.0,17.0)), 2.0+dot(p,float2(11.0,47.0)),3.0+dot(p,float2(41.0,29.0)), 4.0+dot(p,float2(23.0,31.0))))*103.0);
	}
};

Functions f;

float2 iuv = floor(uv);
float2 fuv = frac(uv);

// Generate per-tile transformation
// #if defined (USE_HASH)
	float4 ofa = f.hash4(iuv + float2(0, 0));
	float4 ofb = f.hash4(iuv + float2(1, 0));
	float4 ofc = f.hash4(iuv + float2(0, 1));
	float4 ofd = f.hash4(iuv + float2(1, 1));
/*
#else
	float4 ofa = tex2D(_NoiseTex, (iuv + float2(0.5, 0.5))/256.0);
	float4 ofb = tex2D(_NoiseTex, (iuv + float2(1.5, 0.5))/256.0);
	float4 ofc = tex2D(_NoiseTex, (iuv + float2(0.5, 1.5))/256.0);
	float4 ofd = tex2D(_NoiseTex, (iuv + float2(1.5, 1.5))/256.0);
	#endif
*/


	// Compute the correct derivatives
float2 dx = ddx(uv);
float2 dy = ddy(uv);

	// Mirror per-tile uvs
ofa.zw = sign(ofa.zw - 0.5);
ofb.zw = sign(ofb.zw - 0.5);
ofc.zw = sign(ofc.zw - 0.5);
ofd.zw = sign(ofd.zw - 0.5);

float2 uva = uv * ofa.zw + ofa.xy, dxa = dx * ofa.zw, dya = dy * ofa.zw;
float2 uvb = uv * ofb.zw + ofb.xy, dxb = dx * ofb.zw, dyb = dy * ofb.zw;
float2 uvc = uv * ofc.zw + ofc.xy, dxc = dx * ofc.zw, dyc = dy * ofc.zw;
float2 uvd = uv * ofd.zw + ofd.xy, dxd = dx * ofd.zw, dyd = dy * ofd.zw;

// Fetch and blend
float2 b = smoothstep(blendRatio, 1.0 - blendRatio, fuv);
 

return lerp(lerp(Texture2DSampleGrad(inTex, inTexSampler, uva, dxa, dya), Texture2DSampleGrad(inTex, inTexSampler, uvb, dxb, dyb), b.x),
		   lerp(Texture2DSampleGrad(inTex, inTexSampler, uvc, dxc, dyc), Texture2DSampleGrad(inTex, inTexSampler, uvd, dxd, dyd), b.x), b.y);

```
