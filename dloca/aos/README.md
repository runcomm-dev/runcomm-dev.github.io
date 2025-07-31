package kr.co.touchad.main

import android.Manifest
import android.content.pm.PackageManager
import android.graphics.Point
import android.os.Build
import android.os.Bundle
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.NotificationManagerCompat
import kr.co.touchad.databinding.ActivityMainBinding
import kr.co.touchad.sdk.TouchAdPlatform
import kr.co.touchad.sdk.common.Constants
import kr.co.touchad.main.BasePreference
import kr.co.touchad.sdk.common.utils.Logger
import java.net.URLEncoder


class MainActivity : AppCompatActivity() {

    private var permissionHelper: PermissionHelper? = null

    private lateinit var binding: ActivityMainBinding

    private val isProd = true

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        if (NotificationManagerCompat.from(this).areNotificationsEnabled())
        {
            init()
        }
        else
        {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU)
            {
                checkRequiredPermission(Manifest.permission.POST_NOTIFICATIONS)
            }
        }
    }

    private fun init() {
        //5.ta.runcomm.co.kr
        BasePreference.init(this)
        val saveCid : String = BasePreference.getValue(BasePreference.MBR_ID, "").toString()
        val saveDomain : String = BasePreference.getValue(BasePreference.DOMAIN, Constants.REAL_DOMAIN).toString()
        binding.etDomain.setText(saveDomain)
        binding.etCid.setText(saveCid)

        binding.tvCharging.setOnClickListener {
            if(binding.etCid.text.toString().isEmpty())
            {
                Toast.makeText(this, "고객관리번호를 입력해주세요.", Toast.LENGTH_LONG).show()
            }
//            else if(binding.etDomain.text.toString().isEmpty())
//            {
//                Toast.makeText(this, "도메인을 입력해주세요.", Toast.LENGTH_LONG).show()
//            }
            else
            {
//                if (checkDomain(binding.etDomain.text.toString()))
//                {
//                    TouchAdPlatform.domain = binding.etDomain.text.toString()
//                    BasePreference.put(BasePreference.DOMAIN, TouchAdPlatform.domain)
//
//                    val cid : String = binding.etCid.text.toString()
//                    TouchAdPlatform.openLaBangEarningMenu(this, isProd, cid, "", "")
//                }
//                else
//                {
//                    Toast.makeText(this, "도메인을 올바르게 입력해주세요.", Toast.LENGTH_LONG).show()
//                }
                val cid : String = binding.etCid.text.toString()
                TouchAdPlatform.openLaBangEarningMenu(this, isProd, cid, "M", "1989")
            }
        }

        binding.todayCharging.setOnClickListener {
            if(binding.etCid.text.toString().isEmpty())
            {
                Toast.makeText(this, "고객관리번호를 입력해주세요.", Toast.LENGTH_LONG).show()
            }
//            else if(binding.etDomain.text.toString().isEmpty())
//            {
//                Toast.makeText(this, "도메인을 입력해주세요.", Toast.LENGTH_LONG).show()
//            }
            else
            {
//                if (checkDomain(binding.etDomain.text.toString()))
//                {
//                    TouchAdPlatform.domain = binding.etDomain.text.toString()
//                    BasePreference.put(BasePreference.DOMAIN, TouchAdPlatform.domain)
//
//                    val cid : String = binding.etCid.text.toString()
//                    TouchAdPlatform.openTodayEarningMenu(this, isProd, cid, "", "")
//                }
//                else
//                {
//                    Toast.makeText(this, "도메인을 올바르게 입력해주세요.", Toast.LENGTH_LONG).show()
//                }
                val cid : String = binding.etCid.text.toString()
                TouchAdPlatform.openTodayEarningMenu(this, isProd, cid, "M", "1989")
            }
        }
    }

    private fun checkRequiredPermission(permission: String) {

        permissionHelper = PermissionHelper(arrayOf(permission), this)

        if (permissionHelper!!.checkPermissionInApp()) {
            return
        }

        permissionHelper!!.requestPermission(0, object : PermissionHelper.PermissionCallback {
            override fun onPermissionResult(permissions: Array<String>, grantResults: IntArray?) {
                if (grantResults!!.isNotEmpty()) {
                    var isGranted : Boolean = true
                    for (i in grantResults.indices) {
                        if (grantResults[i] != PackageManager.PERMISSION_GRANTED) {
                            isGranted = false
                            break
                        }
                    }

                    if (!isGranted)
                    {
                        Toast.makeText(applicationContext, "알림 권한을 수락하셔야 기능을 사용하실 수 있습니다.", Toast.LENGTH_SHORT).show()
                    }
                    init()
                }
            }
        })
    }

    override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<String>, grantResults: IntArray) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (permissionHelper != null) permissionHelper!!.onRequestPermissionsResult(requestCode, permissions, grantResults)
    }

    private fun checkDomain(domain: String): Boolean {
        val filter = "^(?:(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?\\.)+[a-zA-Z]{2,}|(?:\\d{1,3}\\.){3}\\d{1,3})(?::([1-9]\\d{0,4}))?\$"
        return domain.matches(filter.toRegex())
    }
}
