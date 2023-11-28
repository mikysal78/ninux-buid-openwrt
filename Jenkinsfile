#!/usr/bin/env groovy

import org.apache.commons.io.FilenameUtils

Map ARCHIVES_PATH = [
    "lamobo_R1": "openwrt/bin/targets/sunxi/cortexa7/openwrt-sunxi-cortexa7-lamobo_lamobo-r1-ext4-sdcard.img.gz",
    "glinet_gl-mt300n-v2": "openwrt/bin/targets/ramips/mt76x8/openwrt-ramips-mt76x8-glinet_gl-mt300n-v2-squashfs-sysupgrade.bin",
    "linksys_wrt3200acm": "openwrt/bin/targets/mvebu/cortexa9/openwrt-mvebu-cortexa9-linksys_wrt3200acm-squashfs-sysupgrade.bin",
    "totolink_X5000R": "openwrt/bin/targets/ramips/mt7621/openwrt-ramips-mt7621-totolink_x5000r-squashfs-sysupgrade.bin",
    "tplink_c2600": "openwrt/bin/targets/ipq806x/generic/openwrt-ipq806x-generic-tplink_c2600-squashfs-sysupgrade.bin",
    "X86_64": "openwrt/bin/targets/x86/64/openwrt-x86-64-generic-ext4-combined-efi.img.gz",
    "raspy_3": "openwrt/bin/target/bcm27xx/bcm2710/openwrt-bcm27xx-bcm2710-rpi-3-squashfs-sysupgrade.img.gz",
    "raspy_4": "openwrt/bin/target/bcm27xx/bcm2711/openwrt-bcm27xx-bcm2711-rpi-4-squashfs-sysupgrade.img.gz"
]

String target_choices = (ARCHIVES_PATH.keySet() as String[]).join(',')

properties([
    parameters([
        booleanParam(name: 'CLEAN_BUILD', defaultValue: true, description: 'deletes contents of the directories /bin and /build_dir.'),
        extendedChoice(defaultValue: "${target_choices}", description: 'Which target to build', multiSelectDelimiter: ',', name: 'TARGETS', quoteValue: false, saveJSONParameterToFile: false, type: 'PT_MULTI_SELECT', value: "${target_choices}", visibleItemCount: 6),
    ])
])

String[] TARGETS =  params.TARGETS.split(',')

// parallel task map
Map tasks = [failFast: false]

TARGETS.each { target ->
    tasks[target] = { ->
        node {
            def UPLOAD_FILE = ARCHIVES_PATH[target]
            def ARCHIVE_NAME = FilenameUtils.getBaseName(UPLOAD_FILE)
            ws("${WORKSPACE}/../ninux-build-openwrt") {
                stage('Checkout') {
                    // Get some code from a GitHub repository
                    checkout scm
                }
                // Mark the code checkout 'stage'....
                tools = load "jenkins.groovy"
                tools.build(target)
                tools.publishArtifact(UPLOAD_FILE)
                tools.githubRelease(UPLOAD_FILE, ARCHIVE_NAME)
            }
        }
    }
}


stage("Parallel builds") {
    parallel(tasks)
}
