# はじめに



## Script

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class LaserPointer : MonoBehaviour {

    [SerializeField]
    private Transform _RightHandAnchor;

    [SerializeField]
    private Transform _LeftHandAnchor;

    [SerializeField]
    private Transform _CenterEyeAnchor;

    [SerializeField]
    private float _MaxDistance = 100.0f;

    [SerializeField]
    private LineRenderer _LaserPointerRenderer;

    public Text text;

    //コントローラ
    private Transform Pointer
    {
        get
        {
            var controller = OVRInput.GetActiveController();
            if(controller == OVRInput.Controller.RTrackedRemote)
            {
                return _RightHandAnchor;
            }else if (controller == OVRInput.Controller.LTrackedRemote)
            {
                return _LeftHandAnchor;
            }

            return _CenterEyeAnchor;
        }

    }


    // Use this for initialization
    void Start () {
		
	}
	
	// Update is called once per frame
	void Update () {
        var pointer = Pointer;
        if(pointer == null || _LaserPointerRenderer == null)
        {
            return;
        }
        //コントローラー位置からRayを飛ばす
        Ray pointerRay = new Ray(pointer.position, pointer.forward);

        //レーザー起点
        _LaserPointerRenderer.SetPosition(0, pointerRay.origin);

        RaycastHit hitInfo;
        if(Physics.Raycast(pointerRay, out hitInfo, _MaxDistance))
        {
            //Rayがヒットしたらそこまで
            _LaserPointerRenderer.SetPosition(1, hitInfo.point);

            //ヒットしたオブジェクトを取得
            GameObject obj = hitInfo.collider.gameObject;
            //ヒットしたオブジェクトのscaleを取得
            Vector3 scale = obj.transform.localScale;

            if (OVRInput.GetDown(OVRInput.Button.PrimaryIndexTrigger))
            {
                Vector3 maxScale = new Vector3(5f,5f,5f);
                if(scale.sqrMagnitude < maxScale.sqrMagnitude)
                {
                    obj.transform.localScale = new Vector3(scale.x + 0.1f, scale.y + 0.1f, scale.z + 0.1f);
                }
                text.text = "down : Trigger";
            }
            else if (OVRInput.GetDown(OVRInput.Button.PrimaryTouchpad))
            {
                Vector3 minScale = new Vector3(0.5f, 0.5f, 0.5f);
                if (scale.sqrMagnitude > minScale.sqrMagnitude)
                {
                    obj.transform.localScale = new Vector3(scale.x - 0.1f, scale.y - 0.1f, scale.z - 0.1f);
                }
                text.text = "down : TouchPad";
            }

        }
        else
        {
            //Rayがヒットしていなかったら向いている方向にMaxDistanceに伸ばす
            _LaserPointerRenderer.SetPosition(1, pointerRay.origin + pointerRay.direction * _MaxDistance);
        }

    }
}

```


# 参考サイト

* [Oculus Goで大自然をかけまわってみる](https://qiita.com/ry-kgy/items/a319c812bfade3d01aa0)